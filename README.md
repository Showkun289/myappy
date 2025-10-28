# app.py
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

# ตัวอย่างสินค้า mock
mock_products = [
    {"shop": "Shop A", "price": 100, "shipping_fee": 30, "type": "delivery"},
    {"shop": "Shop B", "price": 90, "shipping_fee": 40, "type": "delivery"},
    {"shop": "Pickup Store", "price": 85, "shipping_fee": 0, "type": "pickup"},
]

GOOGLE_API_KEY = "YOUR_GOOGLE_MAPS_API_KEY"  # ใส่ API Key

@app.route('/compare', methods=['GET'])
def compare():
    user_location = request.args.get('location')  # เช่น "13.7563,100.5018"
    pickup_location = "13.7460,100.5347"  # สมมุติที่อยู่ Pickup Store

    result = []
    for product in mock_products:
        total_price = product['price'] + product['shipping_fee']
        transport_cost = 0

        if product['type'] == 'pickup':
            # คำนวณค่าเดินทางจาก Google Maps
            url = f"https://maps.googleapis.com/maps/api/directions/json?origin={user_location}&destination={pickup_location}&mode=transit&key={GOOGLE_API_KEY}"
            res = requests.get(url)
            data = res.json()
            if data["routes"]:
                # สมมุติใช้ค่า fare หรือระยะทาง x 2 บาท/km
                try:
                    fare = data["routes"][0]["fare"]["value"]
                    transport_cost = fare
                except:
                    distance = data["routes"][0]["legs"][0]["distance"]["value"] / 1000
                    transport_cost = distance * 2

            total_price += transport_cost

        result.append({
            "shop": product["shop"],
            "price": product["price"],
            "shipping_fee": product["shipping_fee"],
            "transport_cost": round(transport_cost, 2),
            "total_price": round(total_price, 2)
        })

    # เรียงตาม total_price
    result = sorted(result, key=lambda x: x["total_price"])

    return jsonify(result)

if __name__ == '__main__':
    app.run(debug=True)
