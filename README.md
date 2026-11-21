# BookMySeat

A movie ticket booking platform built with Django. Users browse movies, pick a show and seats, pay through Razorpay, and receive a PDF ticket with a verification QR code.

**Live demo:** https://bookmyseat-6ns8.onrender.com/movies/

## Features

- **Movie catalogue:** movies with genres, languages, cast, posters, trailers and release dates.
- **Search and discovery:** title search, filters (genre, language, city, theater, show date, release date, rating, show time), sorting (popularity, newest, rating, price), and pagination.
- **Seat selection:** live seat map with two-minute temporary holds, multi-seat selection, and protection against double booking.
- **Payments:** Razorpay checkout with server-side signature verification, a webhook, refunds for seats that were lost, and retry for failed payments.
- **Tickets:** PDF tickets with a signed QR verification link, download from booking history, and email delivery through a Celery background task with retries.
- **Reviews and ratings:** one review per movie, verified-viewer badges for people who watched, and moderation through reports.
- **Recommendations:** personalised picks from successful bookings and recent views, with a labelled general fallback for new users.
- **Admin dashboard:** revenue (gross, refunded, net), bookings, theater occupancy, movie and theater rankings, peak hours, cancellations, user growth, and CSV export, restricted by permission.

## Tech stack

- Python 3.14, Django 5.2
- SQLite locally, PostgreSQL on Render (via `DATABASE_URL`)
- Razorpay (test mode)
- Celery with Redis for background email (optional)
- ReportLab (PDF) and qrcode
- WhiteNoise for static files, Gunicorn in production

## Getting started (local)

```bash
python -m venv venv
venv\Scripts\activate          # Windows
pip install -r requirements.txt
```

Create a `.env` file in the project root:

```
DJANGO_SECRET_KEY=change-me
DJANGO_DEBUG=True
RAZORPAY_KEY_ID=rzp_test_xxxxxxxx
RAZORPAY_KEY_SECRET=your_test_secret
RAZORPAY_WEBHOOK_SECRET=your_webhook_secret
EMAIL_HOST_USER=you@gmail.com
EMAIL_HOST_PASSWORD=your_16_char_app_password
```

Then set up the database and demo data:

```bash
python manage.py migrate
python manage.py setup_demo_catalog
python manage.py createsuperuser
python manage.py runserver
```

Open http://127.0.0.1:8000/movies/.

### Optional: background email worker

```bash
celery -A bookmyseat worker --loglevel=info --pool=solo
```

Requires Redis on `127.0.0.1:6379`. Without it, tickets still download from the profile.

## Running the tests

```bash
python manage.py test movies
```

## Deployment (Render)

- **Build command:** `pip install -r requirements.txt && python manage.py collectstatic --noinput && python manage.py migrate && python manage.py setup_demo_catalog`
- **Start command:** `gunicorn bookmyseat.wsgi`
- **Environment variables:** `DJANGO_SECRET_KEY`, `DJANGO_DEBUG=False`, `DJANGO_ALLOWED_HOSTS`, `CSRF_TRUSTED_ORIGINS`, `DATABASE_URL`, `SITE_BASE_URL`, the Razorpay keys, and the email credentials.

## Test payments

Use Razorpay's test mode only. The Indian test card is:

- Card: `5267 3181 8797 5449`
- Expiry: any future date
- CVV: any three digits

International cards are declined on this merchant account, so use the Indian card above.

## Project structure

```
bookmyseat/   project settings, URLs, Celery app
movies/       catalogue, seats, payments, tickets, discovery, dashboard
users/        registration, login, profile
templates/    HTML templates
static/       CSS and JavaScript
```

## Notes

- The live demo uses Razorpay **test** mode. No real money is charged.
- Ticket emails need a running Celery worker and Redis, which the free Render plan doesn't include.
