django + vue 前后端分离  跨域请求 OPTIONS  解决方案



1. pip install django-cors-headers

2. settings  INSTALLED_APPS 中注册 ‘corsheaders’

3.  开启

   ```
   # 跨域
   CORS_ORIGIN_ALLOW_ALL = True
   CORS_ALLOW_CREDENTIALS = True
   CORS_ALLOW_HEADERS = ('*', )   # 重要
   ```

4. 中间件中加入

   ```
   'django.contrib.sessions.middleware.SessionMiddleware',
   'corsheaders.middleware.CorsMiddleware',   # 加在session下
   ```

   



中间件自定义拦截器

新建mymiddleware.py

```
from django.utils.deprecation import MiddlewareMixin
from django_redis import get_redis_connection
from .utls import http_response
from django.shortcuts import HttpResponse
from django.http import JsonResponse
import base64


# 拦截器 验证token
class SimpleMiddleware(MiddlewareMixin):
    def process_request(self, request):
        try:
            if request.path == '/users/register/' or request.path == '/users/login/':
                return None
            token = request.META.get("HTTP_TOKEN")
            print(token)
            if not token:
                return JsonResponse({
                    "success": False,
                    "code": 401,
                    "msg": "请重新登录"
                })
            conn = get_redis_connection("default")
            t_b64_decode = base64.b64decode(token.encode()).decode()
            num = int(t_b64_decode[-1])
            r_token = conn.get("user_token:" + t_b64_decode[:-num])
            print(r_token)
            if r_token is None:
                return JsonResponse({
                    "success": False,
                    "code": 401,
                    "msg": "请重新登录"
                })
            elif token != r_token:
                return JsonResponse({
                    "success": False,
                    "code": 402,
                    "msg": "您已在其他地方登录"
                })

        except UnicodeDecodeError:
            return JsonResponse({
                "success": False,
                "code": 401,
                "msg": "请重新登录"
            })
        except Exception as e:
            print(e)
            return JsonResponse({
                "success": False,
                "code": 401,
                "msg": "请重新登录"
            })
```

