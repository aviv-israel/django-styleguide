# Django Styleguide
This style guide based on:
1. [Django styleguide of HackSoft](https://github.com/HackSoftware/Django-Styleguide)
2. [Styleguide for Django API Projects of Paul Hallett](https://github.com/phalt/django-api-domains/)

## Introduction
![](Django%20Styleguide/dads_main.png)


## App Layout
![](Django%20Styleguide/app_layout.png)

* A domain **must** use the following file structure:
```
	-  apis.py - Public functions and access points, presentation logic.
	- interfaces.py - Integrations with other domains or external services.
	- models.py - Object models and storage, simple information logic.
	- services.py - coordination and transactional logic.
```* `Views.py` in  [Django’s pattern](https://docs.djangoproject.com/en/dev/#the-view-layer)  is *explicitly not allowed* in this styleguide.
We only focus on API-based applications. Most logic that used to live in Django’s `views.py` would now be separated into APIs and Services.

## Models
* Models **must not** have any complex functional logic in them.
* Models **should** own informational logic related to them.
* Models **can** have computed properties where it makes sense.
* Models **must not** import services, interfaces, or apis from their own domain or other domains.
* models **can** import _common.model_
* Table dependencies (such as ForeignKeys) **must not** exist across domains. Use a UUID field instead, and have your Services control the relationship between models.
* You **can** use ForeignKeys between tables in one domain. (But be aware that this might hinder future refactoring.)
* Any information in the system **needs** to be the _owner_

_Python 2 /3:_
```
import uuid
from django.db import models
from common.models import BaseModel


class Book(BaseModel):

    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    name = models.CharField(max_length=256)
    publisher = models.CharField(max_length=256)
    author_id = models.UUIDField(default=uuid.uuid4)

    @property
    def name_and_publisher(self):
        return f’{self.name}, {self.publisher}’

	  def __str__(self):
      return "MessageLog %d" % self.id


```

## APIs

* APIs **must be** used as the entry point for all other consumers who wish to use this domain.
* APIs **should** own presentational logic and schema declarations.
* Internal domain-to-domain APIs **should** just be functions.
* You **can** group internal API functions under a class if it makes sense for organisation.
* If you are using a class for your internal APIs, it **must** use the naming convention `MyDomainAPI`.
* Internal functions in APIs **must** use type annotations.
* Internal functions in APIs **must** use keyword arguments.
* You **should** log API function calls.
* All data returned from APIs **must be** serializable.
* APIs **must** talk to Services to get data.
* APIs **must not** talk to Models directly.
* APIs **should** do simple logic like transforming data for the outside world, or taking external data and transforming it for the domain to understand.
* Objects represented through APIs **do not** have to map directly to internal database representations of data.

_Python 3_
```
import logging
import uuid
from typing import Dict  # noqa

from .services import BookService

logger = logging.getLogger(__name__)


class BookAPI:

    @staticmethod
    def get(*, book_id: uuid.UUID) -> Dict:
        logger.info('method "get" called')
        return BookService.get_book(id=book_id)
```

### REST API
[Django REST Framework](https://www.django-rest-framework.org/)  is our framework for building REST API services with Django.
we organise the logic in a domain this way:
* `urls.py`- Router and URL configuration.
* `apis.py` - DRF view functions or view classes.
* `serializers.py` - Serialization for models.

Additional ruling for DRF:
* You **should** serialize all models using DRF serializers.
* You **should not** use the  [ModelMixin](https://www.django-rest-framework.org/api-guide/generic-views/#mixins)  Viewsets as they will tightly couple the data layer with the presentation layer.

### GraphQL API
[Graphene-Django](https://docs.graphene-python.org/projects/django/en/latest/)  is the recommended framework for creating  [GraphQL](https://graphql.org/)  APIs with Django.

When using Graphen-Django, we can organise the logic in a domain this way:
* apis.py - Queries and Mutations.
Additional ruling for Graphene-Django:
* You **should not** tightly link an DjangoObjectType to a Django model as this will tightly couple the data layer with the presentation layer. Instead, use a generic ObjectType.

## Interfaces

* The primary components of Interfaces **should** be functions.
* You **can** group functions under a class if it makes sense for organisation.
* If you are using a class, it **must** use the naming convention MyDomainInterface.
* Functions in Interfaces **must** use type annotations.
* Functions in Interfaces **must** use keyword arguments.
_python 3_
```
import uuid
from typing import Dict, Str  # noqa

# Could be an internal domain or an HTTP API client - we don’t care!
from src.authors.apis import AuthorAPI


# plain example
def update_author_name(*, author_name: Str, author_id: uuid.UUID) -> None:
    AuthorAPI.update_author_name(
        id=author_id,
        name=author_name,
    )


# class example
class AuthorInterface:

    @staticmethod
    def get_author(*, id: uuid.UUID) -> Dict:
        return AuthorAPI.get(id=id)

    @staticmethod
    def update_author_name(
      *,
      author_name: Str,
      author_id: uuid.UUID,
    ) -> None:
        AuthorAPI.update_author_name(
            id=author_id,
            name=author_name,
        )
```

## Services
* The primary components of Services **should** be functions.
* Services **should** own co-ordination and transactional logic.
* You **can** group functions under a class if it makes sense for organisation.
* If you are using a class, it **must** use the naming convention MyDomainService.
* If you are using function, it **must** use the naming convention pattern - _ get_user, create_user

* Functions in services.py **must** use type annotations.
* Functions in services.py **must** use keyword arguments.
* You **should** be logging in services.py.