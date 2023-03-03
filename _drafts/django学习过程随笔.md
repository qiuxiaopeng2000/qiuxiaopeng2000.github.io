

1. 创建django项目：`django-admin startproject projectname`
2. 进入django项目所在文件夹，运行django项目：`python manage.py runserver 0.0.0.0:8000`，8000是项目的端口号.注意：由于在配置django的docker容器时将django的8000号端口映射到了服务器的8000号端口，若不加上端口号可能无法访问该django服务。
3. 若想通过云服务器的公网IP直接访问django服务，还需将公网IP加入settings.py文件中的ALLOWED_HOST配置中。

