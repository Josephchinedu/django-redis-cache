# django-redis-cache

### install redis
```bash
sudo apt-get install redis-server
```
check redis was successfully installed

```bash
redis-cli
```

### Requirements for this project
* django
* djangorestframework
* django-redis
* redis
* loadtest


### setting up cache
in your settings. add this code

```python
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

```

### loadtest
```bash
sudo npm install -g loadtest

```

### views
create a view that fetch from database without cache and compare with cache is use

```python
@api_view(['GET'])
def view_books(request):
 
    products = Product.objects.all()
    results = [product.to_json() for product in products]
    return Response(results, status=status.HTTP_201_CREATED)

```

test it
```bash
loadtest -n 100 -k  http://localhost:8000/store/
```


when cache is use
```python
from django.core.cache import cache
from django.conf import settings
from django.core.cache.backends.base import DEFAULT_TIMEOUT
 
CACHE_TTL = getattr(settings, 'CACHE_TTL', DEFAULT_TIMEOUT)

@api_view(['GET'])
def view_cached_books(request):
    if 'product' in cache:
        # get results from cache
        products = cache.get('product')
        return Response(products, status=status.HTTP_201_CREATED)
 
    else:
        products = Product.objects.all()
        results = [product.to_json() for product in products]
        # store data in cache
        cache.set(product, results, timeout=CACHE_TTL)
        return Response(results, status=status.HTTP_201_CREATED)


```

test the view that has cache feature
```bash
sudo loadtest -n 100 -k  http://localhost:8000/store/cache/
```

