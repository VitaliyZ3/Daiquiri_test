# Task 1

Solved models.py will look like this.
```python
class Product(Model):
    class Meta:
        db_table = "product"
        indexes = [
            models.Index(fields=['category', 'price']),
            models.Index(fields=['category', 'rank']),
            models.Index(fields=['price']), # Added index for price field
        ]
    
    name = models.TextField(_("Name"))
    category = models.ForeignKey(Category, RESTRICT, verbose_name=_("Category"), related_name="products")
    price = models.DecimalField(_("Price"), decimal_places=2, max_digits=10, null=True)
    rank = models.DecimalField(_("Price"), decimal_places=2, max_digits=10, null=True)
    perks = models.ManyToManyField(Perk, verbose_name=_("Perks"), related_name="products")


class Category(Model):
    class Meta:
        db_table = "category"
        name = models.TextField(_("Name"), unique=True)


class Perk(Model):
    class Meta:
        db_table = "perk"
        name = models.TextField(_("Name"), unique=True)



class ProductPerk(models.Model):
    '''
    This model is used to store the relationship and indexes between Product and Perk models
    '''
    product = models.ForeignKey(Product, on_delete=models.SET_NULL, null=True, blank=True)
    perk = models.ForeignKey(Perk, on_delete=models.SET_NULL, null=True, blank=True)

    class Meta:
        db_table = "product_perk"
        indexes = [
            models.Index(fields=['product']),
            models.Index(fields=['perk']),
        ]
```  

Also knowing the fields we will need we can use the function
only() that provide an optimization by returning the fields you need:
```python
Products.object.filter(...some_values).only("list of fiels, that we need return to frontend")
```

Next one is select and prefetch related for solving n+1 problem
```python
products = products.select_related('category').prefetch_related(
            Prefetch('perks', queryset=Perk.objects.only('id', 'name'))
)
```

The last one is testing, you can choose Postman, Jmeter, or Locust. 
I prefer to use Locust, because that one is intuitive, has UI and is easy to learn and launch

Locust example test code
```python
from locust import HttpUser, TaskSet, task, between

class UserBehavior(TaskSet):

    @task
    def filter_products(self):
        params = {
            'category_id': 1,
            'perk_id_list[]': [1, 2, 3],
            'price_from': 100,
            'price_to': 5000,
            'order_by': 'price',
            'page': 1
        }
        self.client.get("/api/products/filter/", params=params)

class WebsiteUser(HttpUser):
    tasks = [UserBehavior]
    wait_time = between(1, 5)
```

And bash command for running it
```bash
locust -f simple_django_test.py --host http://localhost:8000 --users 50 --spawn-rate 2
```