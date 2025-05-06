贴一段源码
```cs
struct attribute {
    const char      *name;
    umode_t         mode;
#ifdef CONFIG_DEBUG_LOCK_ALLOC
    bool            ignore_lockdep:1;
    struct lock_class_key   *key;
    struct lock_class_key   skey;
#endif
};
```

sysfs_attr_init
```cs
/**
 *  sysfs_attr_init - initialize a dynamically allocated sysfs attribute
 *  @attr: struct attribute to initialize
 *
 *  Initialize a dynamically allocated struct attribute so we can
 *  make lockdep happy.  This is a new requirement for attributes
 *  and initially this is only needed when lockdep is enabled.
 *  Lockdep gives a nice error when your attribute is added to
 *  sysfs if you don't have this.
 */

#ifdef CONFIG_DEBUG_LOCK_ALLOC
#define sysfs_attr_init(attr)               \
do {                            \
    static struct lock_class_key __key;     \
                            \
    (attr)->key = &__key;               \
} while (0)
#else
#define sysfs_attr_init(attr) do {} while (0)
#endif
```

```cs
/**
 * struct attribute_group - data structure used to declare an attribute group.
 * @name:   Optional: Attribute group name
 *      If specified, the attribute group will be created in
 *      a new subdirectory with this name.
 * @is_visible: Optional: Function to return permissions associated with an
 *      attribute of the group. Will be called repeatedly for each
 *      non-binary attribute in the group. Only read/write
 *      permissions as well as SYSFS_PREALLOC are accepted. Must
 *      return 0 if an attribute is not visible. The returned value
 *      will replace static permissions defined in struct attribute.
 * @is_bin_visible:
 *      Optional: Function to return permissions associated with a
 *      binary attribute of the group. Will be called repeatedly
 *      for each binary attribute in the group. Only read/write
 *      permissions as well as SYSFS_PREALLOC are accepted. Must
 *      return 0 if a binary attribute is not visible. The returned
 *      value will replace static permissions defined in
 *      struct bin_attribute.
 * @attrs:  Pointer to NULL terminated list of attributes.
 * @bin_attrs:  Pointer to NULL terminated list of binary attributes.
 *      Either attrs or bin_attrs or both must be provided.
 */

struct attribute_group {
    const char      *name;
    umode_t         (*is_visible)(struct kobject *,
                          struct attribute *, int);
    umode_t         (*is_bin_visible)(struct kobject *,
                          struct bin_attribute *, int);
    struct attribute    **attrs;
    struct bin_attribute    **bin_attrs;
};
```

```cs
#define __ATTR(_name, _mode, _show, _store) {               \
    .attr = {.name = __stringify(_name),                \
         .mode = VERIFY_OCTAL_PERMISSIONS(_mode) },     \
    .show   = _show,                        \
    .store  = _store,                       \
}

#define __ATTR_PREALLOC(_name, _mode, _show, _store) {          \
    .attr = {.name = __stringify(_name),                \
         .mode = SYSFS_PREALLOC | VERIFY_OCTAL_PERMISSIONS(_mode) },\
    .show   = _show,                        \
    .store  = _store,                       \
}

#define __ATTR_RO(_name) {                      \
    .attr   = { .name = __stringify(_name), .mode = S_IRUGO },  \
    .show   = _name##_show,                     \
} 

#define __ATTR_RO_MODE(_name, _mode) {                  \
    .attr   = { .name = __stringify(_name),             \
            .mode = VERIFY_OCTAL_PERMISSIONS(_mode) },      \
    .show   = _name##_show,                     \
}  

#define __ATTR_WO(_name) {                      \
    .attr   = { .name = __stringify(_name), .mode = S_IWUSR },  \
    .store  = _name##_store,                    \
}

#define __ATTR_RW(_name) __ATTR(_name, (S_IWUSR | S_IRUGO),     \
             _name##_show, _name##_store)

#define __ATTR_NULL { .attr = { .name = NULL } }
```
