Date: 2020-12-04
Title: DRF Best Practices01: DynamicFieldsModelSerializer 
Tags: DRF, Django, Python
Slug: bp01_DynamicFieldsModelSerializer



DRF Serializer 容易由于字段过多，表间级联查询过多，造成性能较差，可以通过动态选择字段极大提升性能

### 场景
list 需要的字段少于 retrive 字段

```python
class X(ModelViewSet)
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        exclude_fields = ["field1", "field2", "fk_obj__field3" ] 
        if page is not None:
            serializer = self.get_serializer(
                page, many=True, exclude_fields=exclude_fields
            )
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(
            queryset, many=True, exclude_fields=exclude_fields
        )
        return Response(serializer.data)

```


```python



class DynamicFieldsModelSerializer(serializers.ModelSerializer):
    """
    A ModelSerializer that takes an additional `fields` argument that
    controls which fields should be displayed.
    """

    def __init__(self, *args, **kwargs):
        def _remove_fields(fields, field_names):
            if field_names is not None:
                allowed_fields = [field.split("__")[0] for field in field_names]
                # Drop any fields that are not specified in the `fields` argument.
                allowed = set(allowed_fields)
                existing = set(fields)
                for field_name in existing - allowed:
                    fields.pop(field_name)

            childs = defaultdict(list)
            for field in field_names:
                if len(field.split("__")) > 1:
                    childs[field.split("__")[0]].append(
                        "__".join(field.split("__")[1:])
                    )

            for key, value in childs.items():
                if isinstance(fields[key], serializers.ListSerializer):
                    _remove_fields(fields[key].child.fields, value)
                else:
                    if hasattr(fields[key], "fields"):
                        _remove_fields(fields[key].fields, value)

            # Don't pass the 'fields' arg up to the superclass

        def _delete_fields(fields, field_names):
            if field_names is not None:
                allowed_fields = [field.split("__")[0] for field in field_names]
                # Drop any fields that are not specified in the `fields` argument.
                allowed = set(allowed_fields)
                for field_name in allowed:
                    fields.pop(field_name)

            childs = defaultdict(list)
            for field in field_names:
                if len(field.split("__")) > 1:
                    childs[field.split("__")[0]].append(
                        "__".join(field.split("__")[1:])
                    )

            for key, value in childs.items():
                if isinstance(fields[key], serializers.ListSerializer):
                    _delete_fields(fields[key].child.fields, value)
                else:
                    if hasattr(fields[key], "fields"):
                        _delete_fields(fields[key].fields, value)

            # Don't pass the 'fields' arg up to the superclass

        _fields = kwargs.pop("fields", None)
        _exclude_fields = kwargs.pop("exclude_fields", None)

        # Instantiate the superclass normally
        super(DynamicFieldsModelSerializer, self).__init__(*args, **kwargs)
        # assert _fields or _exclude_fields, 'exclude_fields,fields 只支持一个'

        if _fields:
            _remove_fields(self.fields, _fields)

        if _exclude_fields:
            _delete_fields(self.fields, _exclude_fields)
```

### Links
- https://stackoverflow.com/questions/54893891/django-rest-framework-using-dynamicfieldsmodelserializer-for-excluding-serializ
- https://www.django-rest-framework.org/api-guide/serializers/#dynamically-modifying-fields