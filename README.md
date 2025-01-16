# Mpesa_API
## **PLP Academy Practice**


# Mpesa Daraja API Integration with Django

This guide provides a simple explanation of how to integrate the **Mpesa Daraja API** into a Django project. Mpesa is a popular mobile money service in Africa, and the Daraja API allows developers to integrate Mpesa payment functionalities into their applications.

---

## **Why Integrate Mpesa Daraja API?**

1. **Mobile Money Payments**:
   - Mpesa is widely used in Africa, making it an essential payment method for businesses targeting this region.
   - The Daraja API enables seamless integration of Mpesa payments into your Django application.

2. **Real-Time Payments**:
   - The API supports real-time payment notifications, allowing you to update your system immediately when a payment is made.

3. **Security**:
   - Mpesa Daraja API uses secure authentication methods (OAuth) to ensure safe transactions.

4. **Scalability**:
   - The API is designed to handle high transaction volumes, making it suitable for growing businesses.

---

## **Requirements**

To integrate Mpesa Daraja API into your Django project, you need the following dependencies:

### **Dependencies**
Create a `requirements.txt` file with the following packages:

```plaintext
Django==4.2.7
requests==2.31.0
django-environ==0.11.2
python-dotenv==1.0.0
```

- **Django**: The web framework used to build the project.
- **requests**: A Python library for making HTTP requests to the Daraja API.
- **django-environ**: A library for managing environment variables in Django.
- **python-dotenv**: A library to load environment variables from a `.env` file.

---

## **Steps to Integrate Mpesa Daraja API in Django**

### **1. Set Up Your Django Project**
If you haven't already, create a Django project and app:

```bash
django-admin startproject myproject
cd myproject
python manage.py startapp mpesa
```

Add the `mpesa` app to `INSTALLED_APPS` in `settings.py`:

```python
INSTALLED_APPS = [
    ...
    'mpesa',
]
```

### **2. Install Dependencies**
Install the required packages:

```bash
pip install -r requirements.txt
```

### **3. Configure Environment Variables**
Create a `.env` file in the root of your project to store sensitive information like your Daraja API credentials:

```plaintext
MPESA_CONSUMER_KEY=your_consumer_key
MPESA_CONSUMER_SECRET=your_consumer_secret
MPESA_SHORTCODE=your_shortcode
MPESA_PASSKEY=your_passkey
```

Load the environment variables in `settings.py`:

```python
import environ

env = environ.Env()
environ.Env.read_env()

MPESA_CONSUMER_KEY = env('MPESA_CONSUMER_KEY')
MPESA_CONSUMER_SECRET = env('MPESA_CONSUMER_SECRET')
MPESA_SHORTCODE = env('MPESA_SHORTCODE')
MPESA_PASSKEY = env('MPESA_PASSKEY')
```

### **4. Create a Payment View**
In `mpesa/views.py`, create a view to handle Mpesa payments:

```python
import requests
import base64
from datetime import datetime
from django.http import JsonResponse
from django.conf import settings

def get_access_token():
    consumer_key = settings.MPESA_CONSUMER_KEY
    consumer_secret = settings.MPESA_CONSUMER_SECRET
    url = 'https://sandbox.safaricom.co.ke/oauth/v1/generate?grant_type=client_credentials'
    response = requests.get(url, auth=(consumer_key, consumer_secret))
    return response.json().get('access_token')

def initiate_stk_push(request):
    if request.method == 'POST':
        phone_number = request.POST.get('phone_number')
        amount = request.POST.get('amount')

        access_token = get_access_token()
        url = 'https://sandbox.safaricom.co.ke/mpesa/stkpush/v1/processrequest'
        headers = {
            'Authorization': f'Bearer {access_token}',
            'Content-Type': 'application/json'
        }
        timestamp = datetime.now().strftime('%Y%m%d%H%M%S')
        password = base64.b64encode((settings.MPESA_SHORTCODE + settings.MPESA_PASSKEY + timestamp).encode()).decode()
        payload = {
            'BusinessShortCode': settings.MPESA_SHORTCODE,
            'Password': password,
            'Timestamp': timestamp,
            'TransactionType': 'CustomerPayBillOnline',
            'Amount': amount,
            'PartyA': phone_number,
            'PartyB': settings.MPESA_SHORTCODE,
            'PhoneNumber': phone_number,
            'CallBackURL': 'https://yourdomain.com/callback',
            'AccountReference': 'Test',
            'TransactionDesc': 'Test Payment'
        }

        response = requests.post(url, json=payload, headers=headers)
        if response.status_code == 200:
            return JsonResponse(response.json(), status=200)
        else:
            return JsonResponse({'error': 'Payment failed'}, status=400)
```

### **5. Configure URLs**
In `mpesa/urls.py`, add a URL pattern for the payment view:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('initiate-stk-push/', views.initiate_stk_push, name='initiate_stk_push'),
]
```

Include the `mpesa` app URLs in the main `urls.py`:

```python
from django.urls import include, path

urlpatterns = [
    ...
    path('mpesa/', include('mpesa.urls')),
]
```

### **6. Test the Integration**
Run the Django development server:

```bash
python manage.py runserver
```

Use a tool like Postman or a frontend form to send a POST request to `http://127.0.0.1:8000/mpesa/initiate-stk-push/` with the following payload:

```json
{
    "phone_number": "254712345678",
    "amount": "10"
}
```

---

## **Conclusion**

Integrating the Mpesa Daraja API into your Django project enables you to handle mobile money payments seamlessly. By following the steps above, you can set up a payment system that enhances user experience and ensures secure transactions. With Mpesa, you can tap into a widely used payment method in Africa and grow your business.

For more details, refer to the [Mpesa Daraja API Documentation](https://developer.safaricom.co.ke/docs).
watch videos on how to navigate the project [here](https://youtu.be/opCjRG90hXs?list=PLL9VrPrscsUYrTlaDpDQp-JQ2tc-HJI2N) 
