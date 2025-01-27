# Django Rest Framework (DRF)

**Django Rest Framework (DRF)** is a tool in Python used to create APIs easily. It helps applications communicate with each other and provides features like authentication, permissions, and serialization. DRF is widely used for building secure and scalable APIs.

## What is an API?

An **API (Application Programming Interface)** is a way for different software or apps to talk to each other.  
It allows different applications to communicate with each other.

## Why Use DRF?

- **Easy to use** – Reduces the effort needed to build APIs.
- **Supports authentication** – Secure APIs using login, tokens, etc.
- **Serialization** – Converts Django database data into JSON format (used in APIs).
- **Permissions & Throttling** – Controls who can access the API and limits requests.
- **Browsable API** – You can test your API in a web browser without extra tools.

## What is Serialization?

**Serialization** in DRF is the process of converting Django model data (Python objects) into **JSON format** so that it can be used in APIs.  
**Deserialization** is the opposite—converting JSON data back into Python objects before saving them to the database.

## Why is Serialization Needed?

- **APIs use JSON format** – But Django stores data as Python objects. Serialization converts this data into JSON so APIs can send it to users.
- **Converts JSON back to Python** – When an API receives data from a user, it converts JSON back into Python objects before saving it (this is called deserialization).

## Types of Serializers in DRF

### 1. ModelSerializer (Automatic Method)
- Automatically creates a serializer based on a Django model.
- Reduces code because it directly maps model fields to JSON.
- Best for simple CRUD APIs.

### 2. Serializer (Manual Method)
- Provides more control over the serialization process.
- You must define each field manually.
- Best for custom data handling, external APIs, or complex validation.

# Serializers Validation in Django REST Framework (DRF)

Validation in Django REST Framework (DRF) ensures that the data being processed is correct before it is saved or returned. There are multiple ways to validate data in serializers.

---

## **1. Types of Validation in DRF Serializers**

### **a) Field-Level Validation**
You can define validation rules at the field level by using the `validate_<field_name>` method.

```python
from rest_framework import serializers

class BidSerializer(serializers.Serializer):
    bid_id = serializers.IntegerField()
    bid_amount = serializers.DecimalField(max_digits=10, decimal_places=2)
    
    def validate_bid_amount(self, value):
        """Ensure bid amount is greater than zero."""
        if value <= 0:
            raise serializers.ValidationError("Bid amount must be greater than zero.")
        return value
```
✅ This ensures `bid_amount` is greater than **zero**.

---

### **b) Object-Level Validation**
You can validate multiple fields together using the `validate()` method.

```python
class BidSerializer(serializers.Serializer):
    bid_id = serializers.IntegerField()
    bidder_id = serializers.IntegerField()
    bid_amount = serializers.DecimalField(max_digits=10, decimal_places=2)
    
    def validate(self, data):
        """Ensure bidder_id is valid for bidding"""
        if data["bidder_id"] == 0:
            raise serializers.ValidationError("Bidder ID cannot be zero.")
        return data
```
✅ This checks `bidder_id` along with other fields.

---

### **c) Using `validators` Argument in Fields**
You can pass a list of validator functions to fields.

```python
def validate_positive(value):
    """Ensure value is positive."""
    if value <= 0:
        raise serializers.ValidationError("Value must be positive.")

class BidSerializer(serializers.Serializer):
    bid_id = serializers.IntegerField()
    bid_amount = serializers.DecimalField(max_digits=10, decimal_places=2, validators=[validate_positive])
```
✅ This allows **reusability** across multiple fields.

---

### **d) Custom Validation in ModelSerializer**
If you're using **ModelSerializer**, you can override the `validate_<field_name>` or `validate` methods.

```python
from rest_framework import serializers
from myapp.models import Bid

class BidSerializer(serializers.ModelSerializer):
    class Meta:
        model = Bid
        fields = ["bid_id", "bid_amount"]

    def validate_bid_amount(self, value):
        if value < 100:
            raise serializers.ValidationError("Minimum bid amount must be 100.")
        return value
```
✅ This enforces **model-related validation**.

---

### **e) Custom Validators as Classes**
You can define reusable **validator classes**.

```python
from rest_framework import serializers

class MinimumBidValidator:
    def __call__(self, value):
        if value < 100:
            raise serializers.ValidationError("Bid amount must be at least 100.")

class BidSerializer(serializers.Serializer):
    bid_amount = serializers.DecimalField(max_digits=10, decimal_places=2, validators=[MinimumBidValidator()])
```
✅ Useful when a validation rule is used **in multiple places**.

---

## **2. Handling Validation Errors**
When validation fails, DRF raises a `serializers.ValidationError`.

```python
serializer = BidSerializer(data={"bid_id": 1, "bid_amount": -50})
if serializer.is_valid():
    print(serializer.validated_data)
else:
    print(serializer.errors)  # {'bid_amount': ['Bid amount must be greater than zero.']}
```
✅ The `.errors` attribute returns a **dictionary** with field-wise errors.

---

## **3. Validation for Related Models**
If you have **nested serializers**, you can perform validation within those serializers.

```python
class BidderSerializer(serializers.Serializer):
    bidder_id = serializers.IntegerField()
    name = serializers.CharField()

class BidSerializer(serializers.Serializer):
    bid_id = serializers.IntegerField()
    bid_amount = serializers.DecimalField(max_digits=10, decimal_places=2)
    bidder = BidderSerializer()

    def validate_bidder(self, value):
        """Ensure bidder has a valid ID."""
        if value["bidder_id"] <= 0:
            raise serializers.ValidationError("Bidder ID must be positive.")
        return value
```
✅ This validates **nested bidder data**.

---

## **4. Customizing Error Messages**
You can customize error messages for specific fields.

```python
class BidSerializer(serializers.Serializer):
    bid_amount = serializers.DecimalField(
        max_digits=10, decimal_places=2, 
        error_messages={"invalid": "Invalid bid amount format."}
    )
```
✅ This replaces the **default error messages** with custom ones.

---

## **Conclusion**
- **Field-Level Validation** → `validate_<field_name>()`
- **Object-Level Validation** → `validate()`
- **Validators in Fields** → `validators=[...]`
- **Class-Based Validators** → Custom reusable classes
- **Handling Nested Data** → Validate nested serializers
- **Custom Error Messages** → `error_messages` in fields

Would you like an example based on your **Bid** or **BidDocument** model?

