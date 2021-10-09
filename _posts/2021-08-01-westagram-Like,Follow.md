---
title: "TIL26. westagram - Like, Follow"
excerpt: "좋아요와 팔로우 API, related name"
---

## 1. Like model
```python
# postings/models.py
class Like(models.Model):
    pub_date = models.DateTimeField(auto_now_add=True)
    like     = models.BooleanField(default=False)
    post     = models.ForeignKey("Post", on_delete=models.CASCADE)
    user     = models.ForeignKey("users.User", on_delete=models.CASCADE)

    class Meta:
        db_table = "likes"
```
좋아요 기능에는 생성시간, 좋아요, 좋아요한 게시물, 좋아요를 클릭한 유저가 등록되어야 한다. 게시물과 유저는 `Post`와 `User`객체를 참조한다. 프론트에서 좋아요에 대해 boolean 타입으로 요청한다는 가정하에 like 필드를 `Boolean`으로 만들고 초기값은 False로 했다. 

## 2. Like view
```python
# postings/views.py
class LikeView(View):
    @is_user
    def post(self, request):
        data = json.loads(request.body)
        
        try:
            # 게시물이 빈 값인 채로 요청한 경우
            if data["post_id"] == "":
                return JsonResponse({"message":"EMPTY_VALUE"}, status = 400)
	    
            [1] Like.objects.update_or_create(
                post     = Post.objects.get(id = data["post_id"]),
                user     = User.objects.get(id = request.user.id),
                defaults = {"like" : data["like"]}
            )

            return JsonResponse({"message":"SUCCESS"}, status = 201)
        
        except KeyError:
            return JsonResponse({"message":"KEY_ERROR"}, status = 400)

```
POST 메소드로 구현하기 때문에 게시물에 대한 정보(post_id)는 request body에 담겨져 올 수 있지만, url 컨버터를 통해 설정해도 되겠다.

[1] 요청한 user id, post id가 이미 `Post`객체와 `User`객체가 db에 존재한다면, 해당하는 row를 업데이트 한다. 업데이트 하고 싶은 내용은 `defaults`에 작성한다. db에 존재하지 않는다면 새로운 row를 생성한다.

이 때 `like`는 프론트에서 이미 유저의 좋아요/안좋아요를 구분해서 요청한다는 가정 하에 가능한 것이다. 구분없이 요청을 한다면 `delete`메소드로 사용할 수 있다. (아래 follow 기능은 boolean없이 작성해 보았다)


## 3. Follow model
``` python
# postings/models.py
class Follow(models.Model):
    follower = models.ForeignKey("users.User", on_delete=models.CASCADE, related_name="follower")
    followee = models.ForeignKey("users.User", on_delete=models.CASCADE, related_name="followee")

    class Meta:
        db_table = "follows"
```
팔로우하는 사람과 팔로우받는 사람이 있다. 이 둘 모두 `User` 객체를 참조한다. 한 클래스에서 같은 클래스를 참조하는 경우 마이그레이션부터 에러가 발생한다. `User` 객체 입장에서 에러가 발생하는 이유를 생각해보자. 

### related_name을 써야 하는 이유
```
user1 = User.objects.get(id=1)
user1.follow_set.all()
```
`User` 객체는 follow_set 으로 자신을 정참조하는 `Follow` 객체까지는 접근했다. 하지만 그 이상의 정보가 없기 때문에, 자신을 정참조하는 필드 2개 중 무엇에 접근해야 하는지 알 수 없다. 따라서 각각의 필드에 `related_name` 옵션에 참조하는 필드명을 적어주어 `User` 객체가 구분할 수 있도록 한다.

참조
https://fabl1106.github.io/django/2019/05/27/Django-26.-%EC%9E%A5%EA%B3%A0-related_name-%EC%84%A4%EC%A0%95%EB%B0%A9%EB%B2%95.html

## 4. Follow view
```python
# postings/views.py
class FollowView(View):
    @is_user
    def post(self, request):
        data = json.loads(request.body)
        
        # 팔로위 계정이 빈 값인 채로 요청한 경우
        if data["followee_email"] == "":
            return JsonResponse({"message":"EMPTY_VALUE"}, status = 400)
	
        # 팔로위 계정이 db에 없는 경우
        if not User.objects.filter(email = data["followee_email"]).exists():
            return JsonResponse({"message":"INVALID_FOLLOWEE"}, status = 401)
        
        user1          = User.objects.get(id = request.user.id) #팔로우 유저 객체
        user2          = User.objects.get(email = data["followee_email"]) # 팔로위 유저 객체
   [1]  user2_followee = user2.followee.filter(follower_id = user1.id) # 팔로위 유저가 Followee에 있고, 그 row들 중 user1이 팔로우하는 걸 찾아줘
        
   [2]  if user2_followee.exists():
            user2_followee.delete()
            return JsonResponse({"message":"UNFOLLOW"}, status = 201)

   [3]  else:
            Follow.objects.create(
            follower = user1,
            followee = user2
            )
            return JsonResponse({"message":"SUCCESS"}, status = 201)
```
user1은 팔로우하는 사람, user2는 팔로우 당하는 사람이다. user1은 토큰으로 알맞은 계정임이 검증되었다. user2에 대해서는 1) 빈 값 2) db에 없는 경우 2가지 검증을 거친다. 검증 후, `User` 객체에 해당하는 인스턴스를 불러와 user1과 user2로 할당한다.

[1] user2가 `Follow`의 followee에 있는 것들에 접근한다. 그 row들 중 follower가 user1인 것을 필터링한다. use1이 follower이며 그 followee가 user2인 row를 찾았고 user2_followee에 할당한다. 

[2] user2_followee가 `Follow` 테이블에 존재한다면, 이미 팔로우된 상태이기 때문에 삭제시킨다. 

[3] user2_followee가 `Follow` 테이블에 없다면, 현재 언팔로우 상태이기 때문에 user1과 user2를 `Follow` 객체로 생성시킨다.

