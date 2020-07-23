# firstsite_project
나의 첫 장고 프로젝트
# <Django 기본>

1. 가상 환경 구축 및 실행

   ```
   $python -m venv myvenv
   $source myvenv/Scripts/activate
   ```

2. 장고 설치 및 프로젝트 시작

   ```
   $pip install django
   $django-admin startproject "프로젝트 이름"
   ```

    (시작한 뒤에 <u>$cd "프로젝트 이름"</u>으로 디렉토리 위치 바꿔주기)

3. app 생성

   ```
   $python manage.py startapp "앱 이름"
   ```

   이번 실습에서 app 이름은 'my app'으로 하기

4. settings.py에 app생성 알리기

   ```
   <settings.py>
   INSTALLED_APPS = [
       'myapp.apps.MyappConfig', <-- 이거 추가
   ```

5. myapp파일에 templates 폴더를 생성하고, 폴더 안에 home.html파일 생성

6. views.py에 home 함수 정의

   ```
   def home(req):
       return render(req, 'home.html')
   ```

7. urls.py에 경로 알리기(urls.py와 templates, views.py가 다른 폴더에 있기 때문에 import를 하여 가져와야 됨!)

   ```
   import myapp.views
   
   urlpatterns = [
       path('admin/', admin.site.urls),
       path('', myapp.views.home, name = 'home'),
   ]
   ```

8. 서버를 돌려서 지금까지 잘 했나 확인해보기

   ```
   TERMINAL에 
   $pyhton manage.py runserver
   (서버를 끌 때는, 'crtl+c'!)
   ```



# <Models&Admin>

개요: MTV패턴은 DB와 독립되어 있다. 그 중에서, Model은 DB에 어떤 형식으로 Data를 넣을 것인지 정의 하는 역할을 한다. 정의한 데이터는 migration을 통해 전송이 되며, admin 권한을 사용하면 데이터를 삭제하고 생성하며 전반적인 관리가 가능하다.

1. models.py에 class Blog를 생성

   ```
   from django.db import models
    
   class Blog(models.Model):
      title = models.CharField(max_length=200)
      created_at = models.DateTimeField(auto_now_add=True)
      updated_at = models.DateTimeField(auto_now=True)
      body = models.TextField()
   ```

2. TERMINAL에 migration을 사용하여 데이터 전송 요청

   ```
   $ python manange.py makemigrations
   $ python manage.py migrate
   ```

3. admin계정 생성

   ```
   $ python manage.py createsuperuser
   $ Username: 사용자 아이디
   $ Email address: 사용자 이메일 주소
   $ Password: 사용자 비밀 번호. 원래 눈에 안보이니 당황하지 않기.
   ```

   계정 생성 후, runserver한 뒤에 admin 페이지에 접속하여 로그인 해보기.

4. 아까 생성한 blog 데이터를 admin 사이트에서 보이게 하기

   ```
   #admin.py
    
   from django.contrib import admin
   from .models import Blog
    
   admin.site.register(Blog)
   ```

   admin사이트에 blog 데이터 추가 완료. 이제 add를 통해 글을 쓸 수 있다.

   그런데, add를 통해 블로그 글을 작성하면, admin사이트 블로그 목록에 'Blog object(1)' 이런 식으로 나온다. 만약, 내가 입력한 제목이 목록에 뜨게 하고 싶다면,  models.py에서 class Blog를 생성한 자리 아래에

   ```
   class Blog(models.Model):
       def __str__(self):      -->  이거 추가해주기
           return self.title   -->  이거랑
       title = models.CharField(max_length=200)
       created_at = models.DateTimeField(auto_now_add=True)
       updated_at = models.DateTimeField(auto_now=True)
       body = models.TextField()
   ```

   이 함수 추가해주기.

   * 항상 indent 잘 되어 있나 확인해 주기!

5. 위에 생성한 Blog 글 들을 html 화면에 띄우기

   -모델에 있는 Blog class를 import

   -home.html에 나올 객체 blogs의 변수들을 지정

   ```
   #views.py
    
   from django.shortcuts import render
   from .models import Blog
    
   def home(req):
       blogs = Blog.objects
       return render(req, 'home.html', {'blogs' : blogs}
   ```

   여기서, 모델로부터 전달 받은 객체를 "queryset"이라고 하고, 이 "queryset"을 처리하고 정렬해주는 기능을 "method"라고 한다.

6. home.html에 객체(blog)를 작성하고 "queryset method"를 사용하여 어떤 정보를 띄울 것 인지 결정하기.

   ```
   #templates/home.html
    
   {% for blog in blogs.all%}
       <h1> {{blog.title}}</h1> --> 제목
       <p> {{blog.created_at}}</p> --> 생성 날짜
       <p> {{blog.body}}</p> --> 내용 
       <br><br>
   {%endfor%}
   ```

7. runserver을 하여 블로그 글이 잘 보이는지 확인해보기.

# <CRUD 실습(1) - CR>

개요: CRUD란 create, read, update, delete의 약자를 따서 만든 합성어 이다.  create와 read를 통해 admin계정이 아닌 사용자가 직접 '새 글 작성'을 할 수 있는 기능을 추가할 수 있다.

1. 일단, view부터 만들기

   -home, new, create라는 3개의 함수를 생성

   -위에 import 까먹지 않고 잘 했나 꼭 확인하기

   ```
   #views.py
    
   from django.shortcuts import render, redirect
   from .models import Blog
    
   def home(req):
       blogs = Blog.objects
       return render(req, 'home.html', {'blogs' : blogs})
     
   def new(req):
       return render(req, 'new.html')
    
   def create(req):
   ```

   여기서 home은 홈 화면을, new는 새 글을 작성하는 html을 rendering 한다.

   create는 복잡하므로 5번째 단계에서 작성한다.

2. new, created의 url을 추가하기

   -views.py에 생성한 new, created의 함수의 경로를 설정할 차례

   ```
   # urls.py
    
   from django.contrib import admin
   from django.urls import path
   import myapp.views
    
   urlpatterns = [
       path('admin/', admin.site.urls),
       path('', myapp.views.home, name='home'),
       path('new/', myapp.views.new, name='new'),
       path('create/', myapp.views.create, name='create'),
   ]
   ```

3. home.html에 '새 글쓰기' 링크 추가하기

   ```
   #home.html 
   
   <a href="{% url 'new' %}">새 글쓰기</a> --> 이 링크가 추가!
   
   {% for blog in blogs.all%}
   <div>
       <h1> {{blog.title}}</h1>
       <p> {{blog.created_at}}</p>
       <p> {{blog.updated_at}}</p> --> 이 것도 추가되었음(해당 내용과는 상관x)
       <p>{{blog.body}}</p>
   </div>
   {%endfor%}
   ```

   아까 urls.py에서 정의했던 'new' 태그를 통해 ''새 글 쓰기 링크''를 추가해야 추후, 코드의 수정이 쉽다!

4. new.html에 'form' 추가하기

```
# templates/new.html
 
<form action="{% url 'create' %}" method="post">
    {% csrf_token %}
    <div>
      <label for="title">제목</label><br>
      <input type="text" name="title" id="title">
    </div>
    <div>
        <label for="content">내용</label><br>
      <textarea name="body" id="body" cols="30" rows="10"></textarea>
    </div>
    <input type="submit" value="글쓰기">
  </form>
```

5. views.py에 create 함수 작성하기.

   ```
   # views.py
   
   def create(req):
       if(req.method == 'POST'):
           post = Blog()
           post.title = req.POST['title']
           post.body = req.POST['body']
           post.save()
       return redirect('home')
   ```

   - request는 req의 형태로 사용할 수 있다. 다만, 함수 안에서 req을 사용할 거면, req만, request를 사용할 거면 request만 사용하여야 한다.



# <CRUD실습 (2) - UD>

개요: 앞서 실습했던 CR를 통해 글을 작성할 수 있었다. 다음은 UD인데, 이 UD를 통해서 우리는 작성한 글의 내용을 변경하거나, 삭제할 수 있다.



1. ### Detail 페이지 만들기

   -Detail 페이지는 제목만 정렬되어 있는 home화면에서 제목을 클릭할 때 나오는 내용을 담은 페이지 이다.

   1-1. Detail 페이지의 html 파일 생성

   ```
   #detail.html
   
   <h1>제목: {{onepost.title}}</h1>
   <p>작성 시간 : {{onepost.created_at}}</p>
   <p>내용 : {{onepost.body}}</p>
    
   <a href="{% url 'home' %}">홈으로</a>
   ```

   1-2 url 설정

   ```
   	
   path('detail/<int:post_id>',myapp.views.detail,name="detail"),
   
   ```

   1-3 views에 detail 설정

   ```
   def detail(request,post_id):
       onepost=get_object_or_404(Blog,pk=post_id)
       return render(request,'detail.html',{'onepost':onepost})
   ```

   -각 게시물은 각자의 'id' 값을 갖게 되며, 이 값은 해당 게시물의 'pk(primary key)'이다.

   ```
   from django.shortcuts import render,redirect,get_object_or_404
   ```

   꼭! views.py에 redirect와 get_object_or_404를 import 해야 됨!

   -'get_object_or_404'는 id값에 해당하는 object를 가져올 때, 해당 object가 없을 때 404에러를 띄우게 한다.

   1-4 home.html을 수정

   ```
   #home.html 
   
   {% for blog in blogs.all %}
       <h1><a href="{% url 'detail' blog.id %}">제목 : {{blog.title}}</a></h1>
       <p>작성시간 : {{blog.created_at}}</p>
   {% endfor %}
   ```

   수정 이유: 아까도 말했듯이 Home화면에는 제목만과 생성 날짜만 나와야 한다. 자세한 내용은 Detail 페이지에서 출력할 것이기 때문이다.



2. ### Update 기능 추가하기

   -update기능은 쉽게 말해 게시글의 '수정' 기능과 유사하다.

   

   2-1. postedit.html 파일 생성

   -일단 만들어 놓고 내비두기.

   

   2-2. detail.html에서 postedit.html로 넘어갈 수 있도록 설정하기.

   -즉, detail페이지에서 수정 화면으로 넘어갈 수 있게 설정해 놓자는 것

   ```
   <!--detail.html-->
   
   <a href="{% url 'postedit' onepost.id %}">수정</a> --> 이거 추가하기
   ```

   2-3. url 설정

   ```
   path('edit/<int:post_id>',myapp.views.postedit,name="postedit"),
   ```

   2-4. views.py에 postedit 함수 정의

   -아까 detail페이지를 만들 때, 특정 제목에 해당하는 id값을 통해 게시글 내용을 가져온 것 처럼, postedit페이지에도 해당 id값을 통해 내용을 받아와서 원래 내용을 보여줘야 함. (그래야 수정을 하니까)

   ```
   def postedit(request,post_id):
       onepost=get_object_or_404(Blog,pk=post_id)
       return render(request,'postedit.html',{'onepost':onepost})
   ```

   2-5. postedit.html 내용 채우기

   ```
   #postedit.html
    
   <form action="{% url 'postupdate' onepost.id %}" method="POST">
       {% csrf_token %}
       <label for="title">제목</label> <br>
       <input type="text" name="title" id="title" value="{{onepost.title}}">
       <br><br>
    
       <label for="body">내용</label> <br>
       <textarea name="body" id="body" cols="30" rows="20">{{onepost.body}}</textarea> 
       <br><br>
    
       <button type="submit">수정하기</button>
    
   </form>
    
   <a href="{% url 'detail' onepost.id %}">돌아가기</a>
   ```

   2-6.  postupdate url & view 설정

   -위에서 수정하기를 누르면 postupdate함수로 넘어가도록 설정해 놓았으니, url을 설정하여 postupdate의 내용과 url 설정을 해줘야 함.

   ```
   path('postupdate/<int:post_id>',myapp.views.postupdate,name="postupdate"),
   ```

   ```
   def postupdate(request,post_id):
       editpost=get_object_or_404(Blog,pk=post_id)
       editpost.title=request.POST['title']
       editpost.body=request.POST['body']
       editpost.save()
       return redirect('/detail/'+str(post_id))
   ```

   3. ### Delete 기능 추가하기

      -Delete 기능은 삭제 기능이므로, 별다른 html페이지가 필요하지 않다. 간단하게 views.py에서 함수를 정의하고 url을 설정하면 된다.

      3-1.  url 설정

      ```
      path('postdelete/<int:post_id>',myapp.views.postdelete,name="postdelete"),
      ```

      3-2. views.py 에서 함수 정의

      ```
      def postdelete(request,post_id):
          deletepost=get_object_or_404(Blog,pk=post_id)
          deletepost.delete()
          return redirect('home')
      ```

      3-3. detail.html 페이지에서 삭제 버튼 만들어 주기

      ```
      	
      <h1>제목 : {{onepost.title}}</h1>
      <p>작성 시간 : {{onepost.created_at}}</p>
      <p>내용 : {{onepost.body}}</p>
       
      <a href="{% url 'home' %}">홈으로</a>
      <a href="{% url 'postedit' onepost.id %}">수정</a> -->update에서 추가함
      <a href="{% url 'postdelete' onepost.id %}">삭제</a> --> 이거 추가
      
      ```

      
