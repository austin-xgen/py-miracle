[![Build Status](https://travis-ci.org/kolypto/py-miracle.png?branch=master)](https://travis-ci.org/kolypto/py-miracle)

Miracle
=======

Miracle is an ACL for Python that was designed to be well-structuted,
simple yet exhaustive. It uses *permissions* defined on *resources*, and *roles* are granted with the access to them.

To be a universal tool, it does not include any special cases,
does not force you to persist and does not insist on any formats or conventions.

Maximum flexibility and total control. Enjoy! :)

Is a port of [miracle](https://github.com/kolypto/nodejs-miracle/) for NodeJS.






Table of Contents
=================






Reference
=========

Define The Structure
--------------------

### Acl
To start using miracle, instantiate the `Acl` object:

```js
from acl import Acl
acl = Acl()
```

The `Acl` object keeps track of your *resources* and *permissions* defined on them, handles *grants* over *roles* and
provides utilities to manage them. When configured, you can check the access against the defined state.

### Create

Methods from this section allow you to build the *structure*: list of roles, resources and permissions.

It's not required that you have the structure defined before you start granting the access: the `grant()` method
implicitly creates all resources and permissions that were not previously defined.

Start with defining the *resources* and *permissions* on them, then you can grant a *role* with the access to some
permissions on a resource.

For roles, resources & permissions, any hashable objects will do.

#### `add_role(role)`
Define a role.

* `role`: the role to define.

The role will have no permissions granted, but will appear in `list_roles()`.

```js
acl.add_role('admin')
acl.list_roles()  # -> {'admin'}
```

#### `add_resource(resource)`
Define a resource.

* `resources`: the resource to define.

The resource will have no permissions defined but will list in `list_resources()`.

```js
acl.add_resource('blog')
acl.list_resources()  # -> {'blog'}
```

#### `add_permission(resource, permission)`
Define a permission on a resource.

* `resource`: the resource to define the permission on.
    Is created if was not previously defined.
* `permission`: the permission to define.

The defined permission is not granted to anyone, but will appear in `list_permissions(resource)`.

```js
acl.add_permission('blog', 'post')
acl.list_permissions('blog')  # -> {'post'}
```

#### `add(structure)`
Define the whole resource/permission structure with a single dict.

* `structure`: a dict that maps resources to an iterable of permissions.

```js
acl.add({
    'blog': ['post'],
    'page': {'create', 'read', 'update', 'delete'},
})
```

### Remove
#### `remove_role(role)`
Remove the role and its grants.

* `role`: the role to remove.

```js
acl.remove_role('admin')
```

#### `remove_resource(resource)`
Remove the resource along with its grants and permissions.

* `resource`: the resource to remove.

```js
acl.remove_resource('blog')
```

#### `remove_permission(resource, permission)`
Remove the permission from a resource.

* `resource`: the resource to remove the permission from.
* `permission`: the permission to remove.

The resource is not implicitly removed: it remains with an empty set of permissions.

```js
acl.remove_permission('blog', 'post')
```

### List

#### `list_roles()`
Get the set of defined roles.

```js
acl.list_roles()  # -> {'admin', 'anonymous', 'registered'}
```

#### `list_resources()`
Get the set of defined resources, including those with empty permissions list.

```js
acl.list_resources()  # -> {'blog', 'page', 'article'}
```

#### `list_permissions(resource)`
Get the set of permissions for a resource.

* `resource`: the resource to get the permissions for.

```js
acl.list_permissions('page')  # -> {'create', 'read', 'update', 'delete'}
```

#### `list()`
Get the *structure*: hash of all resources mapped to their permissions.

Returns a dict: `{ resource: set(permission,...), ... }`.

```js
acl.list()  # -> { blog: {'post'}, page: {'create', ...} }
```

### Export and Import
The `Acl` class is picklable:

```js
acl = miracle.Acl()
save = acl.__getstate__()

#...

acl = miracle.Acl()
acl.__setstate__(save)
```



Grant Permissions
-----------------

### grant(role, resource, permission)
Grant a permission over resource to the specified role.

* `role`: The role to grant the access to
* `resource`: The resource to grant the access over
* `permission`: The permission to grant with

Roles, resources and permissions are implicitly created if missing.

```js
acl.grant('admin', 'blog', 'delete')
acl.grant('anonymous', 'page', 'view')
```

### revoke(role, resource, permission)
Revoke a permission over a resource from the specified role.

```js
acl.revoke('anonymous', 'page', 'view')
acl.revoke('user', 'account', 'delete')
```



Authorize
---------

### check(role, resource, permission)
Test whether the given role has access to the resource with the specified permission.

* `role`: The role to check
* `resource`: The protected resource
* `permission`: The required permission

Returns a boolean.

```js
acl.check('admin', 'blog') # True
acl.check('anonymous', 'page', 'delete') # -> False
```

### check_any(roles, resource, permission)
Test whether *any* of the given roles have access to the resource with the specified permission.

* `roles`: An iterable of roles.

When no roles are provided, returns False.

### check_all(roles, resource, permission)
Test whether *all* of the given roles have access to the resource with the specified permission.

* `roles`: An iterable of roles.

When no roles are provided, returns False.


Show Grants
-----------

### which(role)
Collect grants that the provided role has:

```js
acl.which('admin')  # -> { blog: {'post'} }
```

### which_any(roles)
Collect grants that any of the provided roles have (union).

```js
acl.which(['anonymous', 'registered'])  # -> { page: ['view'] }
```

### which_all(roles)
Collect grants that all of the provided roles have (intersection):

```js
acl.which(['anonymous', 'registered'])  # -> { page: ['view'] }
```

### show()
Get all current grants.

Returns a dict  `{ role: { resource: set(permission) } }`.

```js
acl.show()  # -> { admin: { blog: ['post'] } }
```
