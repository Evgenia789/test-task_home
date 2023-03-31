# 1
```python
@api_view(['POST'])
def create_entity(request):
    serializer = EntitySerializer(data=request.data)
    try:
        serializer.is_valid(raise_exception=True)
    except ValidationError as e:
        return JsonResponse(e.detail, status=status.HTTP_400_BAD_REQUEST)
    serializer.save(modified_by=request.user)
    return JsonResponse(serializer.data, status=status.HTTP_201_CREATED)
```

# 2, 3   
добавление метода to_internal_value - решение второго вопроса   
добавление метода create и изменение properties - решение третьего вопроса
```python
class EntitySerializer(serializers.ModelSerializer):
    properties = serializers.ListField(child=serializers.DictField(child=serializers.CharField()))

    class Meta:
        model = Entity
        fields = ('properties',)
    
    def to_internal_value(self, data):
        value_key = 'data[value]'
        value = data.get(value_key)
        data.pop(value_key, None)
        validated_data = super().to_internal_value(data)   
        validated_data['value'] = value 
        return validated_data
    
    def create(self, validated_data):
        properties_data = validated_data.pop('properties', [])
        entity = Entity.objects.create(**validated_data)
        for property_data in properties_data:
            property_obj, _ = Property.objects.get_or_create(**property_data)
            entity.properties.add(property_obj)
        return entity
```
