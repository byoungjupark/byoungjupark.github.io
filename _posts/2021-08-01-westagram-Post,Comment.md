---
title: "TIL25. westagram - Post, Comment"
excerpt: "게시물과 댓글 생성,조회 API"
---

## 1. Post model
```python
# postings/models.py
class Post(models.Model):
    pub_date = models.DateTimeField(auto_now_add=True)
    content  = models.TextField(max_length=4000, null=True)
    user     = models.ForeignKey("users.User", on_delete=models.CASCADE)
    
    class Meta:
        db_table = "posts"

class Image(models.Model):
    image = models.CharField(max_length=2000)
    post  = models.ForeignKey("Post", on_delete=models.CASCADE)

    class Meta:
        db_table = "images"
```
게시물 등록에 필요한 생성시간, 게시물 내용, 유저, 게시물이 있다. 게시물 내용없이 등록할 수도 있기 때문에 content는 null을 허용한다. 여러 개의 이미지를 포스트할 수도 있기 때문에 `Image` 클래스를 생성하고 `Post` 객체를 참조하도록 했다. (1:N 관계)

이미지를 `ImageField`로 설정하면 객체 생성시 `TypeError: Object of type ImageFieldFile is not JSON serializable` 에러가 발생한다. `ImageField`나 `FileField`는 db에 특정해놓은 경로에 파일 형태로 저장한다. 파일은 Json 형식으로 전환이 불가능하다.

## 2. Post View
### 2-1 게시물 등록
```python
# postings/views.py
class PostView(View):
    @is_user
    def post(self, request):
        data = json.loads(request.body) 
        
        try: 
            # 이미지없이 게시물을 등록할 경우
            if data["images"] == "":
                return JsonResponse({"message":"EMPTY_VALUE"}, status = 400)

            post = Post.objects.create(
                content = data["content"],
                user    = User.objects.get(id = request.user.id)
            )
            
	    # 여러 개의 이미지를 등록할 수도 있다
            for image in data["images"]: 
                Image.objects.create(
                    image = image,
                    post  = Post.objects.get(id = post.id)
                )
        
            return JsonResponse({"message":"SUCCESS"}, status = 201)

        except KeyError:
            return JsonResponse({"message":"KEY_ERROR"}, status = 400)
```
* 이미지 : 여러 개의 이미지를 등록할 경우, value가 리스트 형태로 온다는 가정 하에 for문으로 db에 각각의 row를 생성한다.

### 2-2 게시물 보기
```python
# postings/views.py
class PostView(View):
    @is_user  
    def get(self, request):
        posts   = Post.objects.all()
        images  = Image.objects.all()
        results = []
        
        for post in posts:
            results.append(  
                {        
                    "pub_date" : post.pub_date,  
                    "content"  : post.content,
                    "user"     : post.user.name,
                    "image"    : [image.image for image in images.filter(post_id = post.id)]                
                }
            )
 
        return JsonResponse({"results":results}, status = 200)
```

## 3. Comment model
```python
# postings/models.py
class Comment(models.Model):
    pub_date = models.DateTimeField(auto_now_add=True)
    comment  = models.CharField(max_length=2000)
    post     = models.ForeignKey("Post", on_delete=models.CASCADE)
    user     = models.ForeignKey("users.User", on_delete=models.CASCADE)

    class Meta:
        db_table = "comments"
```
댓글을 등록하려면 생성시간, 댓글, 댓글이 달린 게시물, 작성자가 필요하다. 게시물과 작성자는 각각 `Post`와 `User`객체를 참조하며 1:N 관계이다.

## 4. Comment view
### 4-1 댓글 등록
```python
# postings/views.py
class CommentView(View):
    @is_user
    def post(self, request):
        data = json.loads(request.body)
        
        try:
            # 댓글이나 게시물 없이 댓글을 등록하려는 경우
            if data["comment"] == "" or data["post_id"] == "":
                return JsonResponse({"message":"EMPTY_VALUE"}, status = 400)
	    
            # 요청한 게시물이 db에 없는 경우
            if not Post.objects.filter(id = data["post_id"]).exists():
                return JsonResponse({"message":"INVALID_POST"}, status = 400)

            Comment.objects.create(
                comment = data["comment"],
                post    = Post.objects.get(id = data["post_id"]),
                user    = User.objects.get(id = request.user.id)
            )

            return JsonResponse({"message":"SUCCESS"}, status = 201)
        
        except KeyError:
            return JsonResponse({"message":"KEY_ERROR"}, status = 400)
```
* 에러 : 유저에 대한 검증은 데코레이터를 통해 마쳤다. 댓글과 게시물에 대한 에러에는 1) 댓글과 게시물 없이 요청한 경우 2) 요청한 게시물이 db `posts` 테이블에 없는 경우이다.

* 댓글 생성 : 에러 조건에 해당되지 않으면 이제 댓글을 생성할 수 있다. comment는 요청한 value를 넣어주고, post에는 `Post` 객체에 해당하는 id와 매치시켜 넣어준다. user에는 토큰 복호화로 나온 id가 `User` 객체 id와 일치하는 것을 찾아 넣어준다.

### 4-2 특정 게시물에 대한 댓글 보기 
```python
# postings/views.py
class CommentView(View):
    def get(self, request, post_id): # 특정 게시물은 postings/urls.py에서 컨버터를 이용하여 post_id의 인자로 넣을 수 있다
        try:
            # 요청한 게시물이 db에 없는 경우
            if not Post.objects.filter(id = post_id).exists():
                return JsonResponse({"message":"INVALID_POST"}, status = 400)

            post     = Post.objects.get(id = post_id) # 요청한 게시물을 Post 객체에서 찾아 post 변수로 할당
            comments = Comment.objects.filter(post_id = post.id) # 그 게시물에 대한 댓글만 찾아주기
            results  = []

            for comment in comments:
                results.append(
                    {
                        "pub_date" : comment.pub_date,
                        "comment"  : comment.comment,
                        "user"     : comment.user.name
                    }
                )
            
            return JsonResponse({"results":results}, status = 200)
        
        except KeyError:
            return JsonResponse({"message":"KEY_ERROR"}, status = 400)
```
처음 특정 게시물에 대한 처리를 할 때, request 시 post id를 담아서 요청한다고 가정하고 코드를 작성하였다. 하지만 request GET 메소드일 경우, request body에 내용을 담지 않는다고 한다. 대신 아래와 같이 url 컨버터를 사용할 수 있다.

```python
# postings/urls.py
urlpatterns = [
   path("/<int:post_id>/comment", CommentView.as_view()),
]
```
컨버터는 `<type : name>` 형식으로 되어있다.
* <int:post_id> : `post_id`가 정수이면 매치되어 해당 뷰로 이동시킨다.
예를 들어 요청 url이 postings/1/comment 라면, 매치된 1은 `CommentView`의 키워드 인자로 할당된다. `def get(self, request, post_id=1)` 
