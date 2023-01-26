# Modifying Our User Serializer

We discovered a couple of errors in the code we used in class for the user serializer. Here's how we need to change it:

```diff
# users/serializers.py
from rest_framework import serializers
from .models import CustomUser

class CustomUserSerializer(serializers.Serializer):
    id = serializers.ReadOnlyField()
    username = serializers.CharField(max_length=200)
    email = serializers.CharField(max_length=200)
+   password = serializers.CharField(write_only = True)
    
    def create(self, validated_data):
-       return CustomUser.objects.create(**validated_data)
+       return CustomUser.objects.create_user(**validated_data)
```
