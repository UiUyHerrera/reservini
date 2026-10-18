# Reservini

Online booking for small businesses: barbers, private tutors, clinics. The owner sets up their services and opening hours, shares the link to their page, and clients pick a free slot without having to message them.

The app comes in English and Spanish, works in any time zone and supports twelve currencies, Uruguayan peso included.

> First time using it? The [quick start guide](docs/GUIA.md) (in Spanish) walks through it step by step with screenshots.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/capturas/reservar-oscuro.png">
  <img src="docs/capturas/reservar.png" alt="Booking page where a client picks a day and time for a haircut">
</picture>

## Features

- Accounts for business owners, with sign-up and login.
- A business profile with a public address, time zone, currency and weekly opening hours. Hours are set day by day with a switch and time menus, or typed in as text.
- Services with a length and a price that can be edited, hidden or deleted.
- A public page with the free slots for the next two weeks, filters by day and time of day, and a check so nobody takes a slot that's already booked.
- A confirmation email when booking and a reminder on the morning of the appointment.
- A dashboard with upcoming bookings. When the owner cancels one, the client gets an email.
- A Free plan with 20 bookings a month and a Pro plan with no limit. Switching plans is a demo and charges nothing.
- Light and dark mode.

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/capturas/panel-oscuro.png">
    <img src="docs/capturas/panel.png" width="49%" alt="Business dashboard with the plan, upcoming bookings and services">
  </picture>
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/capturas/horarios-oscuro.png">
    <img src="docs/capturas/horarios.png" width="49%" alt="Opening hours editor, one day at a time">
  </picture>
</p>

## Built with

Python and Flask, with SQLAlchemy on SQLite in development and PostgreSQL in production. Flask-Login handles sessions, Flask-WTF forms and CSRF, Flask-Limiter slows down repeated attempts, and Flask-Babel the translations. The frontend is plain HTML and CSS, with a bit of JavaScript for the hours editor and the booking page filters. Tests use pytest.

## Running it locally

You need Python 3.12 or newer.

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements-dev.txt
copy .env.example .env
```

On macOS or Linux, activate the environment with `source .venv/bin/activate` and copy the file with `cp`.

Set a long random `SECRET_KEY` in `.env`. You can generate one with:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

Then start the server. The database is created on the first run:

```bash
flask run
```

It runs at http://127.0.0.1:5000. Emails are only sent when the `MAIL_*` settings in `.env` point to an SMTP server; otherwise the app logs that it skipped them.

## Tests

```bash
python -m pytest
```

There are 83 tests, and GitHub Actions runs them on every push.

## Translations

Texts are written in English in the code, and the Spanish translation lives in `reservini/translations/es/LC_MESSAGES/messages.po`. After changing texts:

```bash
pybabel extract -F babel.cfg -k _l -o messages.pot .
pybabel update -i messages.pot -d reservini/translations
pybabel compile -d reservini/translations
```

## Reminders

`flask send-reminders` sends the reminders that are due. You can run it as often as you like: it never sends the same one twice.

In production, Vercel calls `/tasks/send-reminders` once a day. The route only answers when the request carries the `CRON_SECRET` value.

## Deploying

It's ready for Vercel with a PostgreSQL database on Neon:

1. Import the repository in Vercel.
2. Connect a Neon database from the Storage tab, which adds `DATABASE_URL`.
3. Add `SECRET_KEY`, `CRON_SECRET`, `TRUST_PROXY=1` and, to send emails, the `MAIL_*` values.

Tables are created on the first start. `api/index.py` is the Vercel entry point, and `vercel.json` sets the region and the reminders cron.
