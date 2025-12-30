# PART 1 — CODE REVIEW & DEBUGGING

 # What Was Wrong?
 Technical Issues
1. No Input Validation
2. No Try–Catch / Error Handling
3. No Transactions → Can create partial data
4. SKU must be unique but not validated
5. Price may be invalid format
6. Multiple commit() calls hurt performance
7. Assumes a product belongs to only ONE warehouse
8. Does not return HTTP status codes
9. No optional field handling

---

#Production Impact

No validation | Corrupted data
No uniqueness | Duplicate SKU
No transaction | Half saved records
No handling | System crashes
Performance bad | Slow API
Wrong logic | Wrong stock data

---

# Corrected Final Code
from flask import request
from app import app, db
from models import Product, Inventory

@app.route('/api/products', methods=['POST'])
def create_product():
    data = request.json

    required = ['name', 'sku', 'price', 'warehouse_id', 'initial_quantity']
    missing = [f for f in required if f not in data]

    if missing:
        return {"error": f"Missing fields: {missing}"}, 400

    # SKU must be unique
    if Product.query.filter_by(sku=data['sku']).first():
        return {"error": "SKU must be unique"}, 409

    # Validate price
    try:
        price = float(data['price'])
        if price <= 0:
            return {"error": "Price must be positive"}, 400
    except:
        return {"error": "Invalid price format"}, 400

    try:
        with db.session.begin():

            product = Product(
                name=data['name'],
                sku=data['sku'],
                price=price
            )
            db.session.add(product)
            db.session.flush()  # Get product.id

            inventory = Inventory(
                product_id=product.id,
                warehouse_id=data['warehouse_id'],
                quantity=data['initial_quantity']
            )
            db.session.add(inventory)

        return {
            "message": "Product created successfully",
            "product_id": product.id
        }, 201

    except Exception as e:
        db.session.rollback()
        return {"error": str(e)}, 500
