# Sam's Django Template

## Quick start

Install uv:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Create a new Django project:

```bash
uvx copier copy https://github.com/SamEdwardes/sams-django-starter mynewapp
cd newapp
```

Install [just](https://github.com/casey/just?tab=readme-ov-file#installation):

```bash
brew install just
```

Bootstrap the project (will install depdencies, create virtual environment, etc.)

```bash
just init
```

Run django in development mode:

```bash
open http://localhost:8000
just dev
```

## Tech stack

- Core logic
  - Async django
  - Background tasks with [django-tasks](https://github.com/realOrangeOne/django-tasks)
  - Logging with [logfire](https://logfire.pydantic.dev/docs/)
  - Authentication with [django-allauth](https://docs.allauth.org/en/latest/)
- UI
  - Server side interactions with [htmx](https://htmx.org/docs/)
  - Client side interactions with [alpine.js](https://alpinejs.dev/start-here)
  - Components with [django-cotton](https://django-cotton.com)
  - Re-usable partials with [django-template-partials](https://github.com/carltongibson/django-template-partials/)
  - CSS styling [tailwindcss](https://tailwindcss.com/docs)
