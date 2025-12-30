#Endpoint:
#GET /api/companies/{company_id}/alerts/low-stock


from flask import jsonify
from datetime import datetime, timedelta
from app import app, db
from models import Product, Warehouse, Inventory, Supplier, ProductSupplier, Sales

@app.route('/api/companies/int:company_id
/alerts/low-stock', methods=['GET'])
def low_stock_alerts(company_id):

products = db.session.query(
    Product.id,
    Product.name,
    Product.sku,
    Warehouse.id.label("warehouse_id"),
    Warehouse.name.label("warehouse_name"),
    Inventory.quantity,
    Product.threshold
).join(Inventory).join(Warehouse).filter(
    Warehouse.company_id == company_id
).all()

alerts = []

for p in products:

    if p.threshold is None:
        continue

    if p.quantity > p.threshold:
        continue

    recent_sales = db.session.query(Sales).filter(
        Sales.product_id == p.id,
        Sales.created_at >= datetime.now() - timedelta(days=30)
    ).count()

    if recent_sales == 0:
        continue

    supplier = db.session.query(Supplier).join(ProductSupplier).filter(
        ProductSupplier.product_id == p.id
    ).first()

    avg_daily_sale = max(1, recent_sales / 30)
    days_left = int(p.quantity / avg_daily_sale)

    alerts.append({
        "product_id": p.id,
        "product_name": p.name,
        "sku": p.sku,
        "warehouse_id": p.warehouse_id,
        "warehouse_name": p.warehouse_name,
        "current_stock": p.quantity,
        "threshold": p.threshold,
        "days_until_stockout": days_left,
        "supplier": {
            "id": supplier.id if supplier else None,
            "name": supplier.name if supplier else None,
            "contact_email": supplier.email if supplier else None
        }
    })

return jsonify({
    "alerts": alerts,
    "total_alerts": len(alerts)
}), 200


---

# Edge Cases Handled Here:
✔ No supplier  
✔ No threshold  
✔ No stock  
✔ No recent sales  
✔ Multiple warehouses handled  

---

# Business Smartness:
✔ Prioritizes real danger stock  
✔ Reduces noise  
✔ Helps company reorder fast  
✔ Predicts future risk
