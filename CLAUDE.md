# CLAUDE.md

This document describes the technology stack and architectural patterns for this Django project. When working with this codebase, please follow these conventions and utilize the specified technologies appropriately.

## About the project

@README.md

## Tech Stack Overview

### Core Logic

#### Async Django

- This project uses Django's async capabilities for improved performance
- Use `async def` for views that perform I/O operations (database queries, API calls, file operations)
- Utilize `sync_to_async()` and `async_to_sync()` decorators when mixing sync/async code
- Prefer async database operations with `aget()`, `acreate()`, `afilter()`, etc.

```python
# Preferred async view pattern
async def my_view(request):
    user = await User.objects.aget(pk=request.user.pk)
    data = await some_async_operation()
    return render(request, 'template.html', {'user': user, 'data': data})
```

#### Background Tasks - django-tasks

- Use django-tasks for background job processing
- Tasks should be defined as async functions when possible
- Utilize task scheduling for delayed execution
- Handle task failures gracefully with retries

```python
from django_tasks import task

@task
async def process_user_data(user_id: int):
    user = await User.objects.aget(pk=user_id)
    # Process user data asynchronously
    pass
```

#### Logging - Logfire

- Use Logfire for structured logging and observability
- Include contextual information in log entries
- Utilize different log levels appropriately (debug, info, warning, error)
- Add tracing for performance monitoring

```python
import logfire

logfire.info("User action completed", user_id=user.id, action="profile_update")
```

#### Authentication - django-allauth

- All authentication flows should use django-allauth
- Customize allauth templates to match project styling
- Utilize allauth's social authentication providers when needed
- Extend allauth adapters for custom authentication logic

### UI Stack

#### Server-Side Interactions - HTMX

- Use HTMX for dynamic page updates without full page reloads
- Implement HTMX patterns: `hx-get`, `hx-post`, `hx-target`, `hx-swap`
- Return partial HTML responses for HTMX requests
- Use HTMX events for client-side coordination

```html
<div hx-get="/api/user-stats" hx-target="#stats-container" hx-trigger="every 30s">
    <div id="stats-container">Loading...</div>
</div>
```

```python
# HTMX-friendly view
async def user_stats_partial(request):
    if request.headers.get('HX-Request'):
        # Return partial template for HTMX
        return render(request, 'partials/user_stats.html', context)
    # Return full page for direct access
    return render(request, 'user_stats.html', context)
```

#### Client-Side Interactions - Alpine.js

- Use Alpine.js for client-side reactivity and DOM manipulation
- Keep Alpine.js code minimal and focused on UI interactions
- Utilize Alpine.js directives: `x-data`, `x-show`, `x-if`, `x-for`
- Combine with HTMX for seamless server-client interactions

```html
<div x-data="{ open: false }">
    <button @click="open = !open">Toggle Menu</button>
    <div x-show="open" x-transition>Menu Content</div>
</div>
```

#### Components - django-cotton

- Create reusable UI components using django-cotton
- Follow component-based architecture principles
- Pass data through component attributes
- Keep components focused and single-purpose

```html
<!-- components/user_card.html -->
<c-vars title name avatar_url />
<div class="bg-white rounded-lg shadow p-4">
    <img src="{{ avatar_url }}" alt="{{ name }}" class="w-12 h-12 rounded-full">
    <h3 class="text-lg font-semibold">{{ title }} {{ name }}</h3>
</div>
```

```html
<!-- Usage -->
<c-user-card title="Dr." name="Jane Smith" avatar_url="/static/images/jane.jpg" />
```

#### Template Partials - django-template-partials

- Use template partials for reusable template fragments
- Ideal for HTMX partial updates and component composition
- Keep partials focused on specific UI sections

```html
<!-- partials/notification.html -->
<div class="bg-blue-100 border border-blue-400 text-blue-700 px-4 py-3 rounded">
    {{ message }}
</div>
```

```python
from template_partials.utils import render_partial

# Return partial for HTMX update
return render_partial(request, 'partials/notification.html', {'message': 'Success!'})
```

#### Styling - Tailwind CSS

- Use Tailwind utility classes for all styling
- Follow mobile-first responsive design principles
- Utilize Tailwind's component layer for reusable styles
- Keep custom CSS minimal, prefer Tailwind utilities

```html
<div class="bg-white shadow-md rounded-lg p-6 max-w-md mx-auto">
    <h2 class="text-2xl font-bold text-gray-800 mb-4">Card Title</h2>
    <p class="text-gray-600 leading-relaxed">Card content here.</p>
</div>
```

## Integration Patterns

### HTMX + Alpine.js

- Use HTMX for server communication, Alpine.js for client-side state
- Alpine.js can react to HTMX events using `@htmx:afterRequest`
- Store temporary UI state in Alpine.js, persistent state on server

### Cotton Components + Partials

- Use Cotton for complete components, partials for fragments
- Cotton components can include multiple partials
- Partials are perfect for HTMX target updates

### Async Views + Background Tasks

- Use async views for immediate responses
- Delegate heavy processing to background tasks
- Provide real-time updates via HTMX polling or WebSockets

## Development Guidelines

### File Organization

```bash
bash
templates/
├── base.html
├── components/          # django-cotton components
│   ├── user_card.html
│   └── navigation.html
├── partials/           # django-template-partials
│   ├── notifications.html
│   └── form_errors.html
└── pages/              # Full page templates
    ├── dashboard.html
    └── profile.html
```

### Performance Considerations

- Use async views for I/O-bound operations
- Implement database query optimization (select_related, prefetch_related)
- Cache frequently accessed data
- Use HTMX for partial page updates to reduce bandwidth
- Leverage background tasks for expensive operations

### Testing

- Write async tests for async views using `pytest-asyncio`
- Test HTMX interactions with proper headers
- Mock background tasks in unit tests
- Use django-cotton's testing utilities for component tests

### Security

- Utilize django-allauth's built-in security features
- Validate HTMX requests to prevent CSRF attacks
- Sanitize user input in Alpine.js data bindings
- Use Django's built-in protections (CSRF, XSS, etc.)

## Common Patterns

### Dynamic Form Handling

```python
async def dynamic_form_view(request):
    if request.method == 'POST':
        form = MyForm(request.POST)
        if form.is_valid():
            await form.asave()
            if request.headers.get('HX-Request'):
                return render_partial(request, 'partials/success.html')
            return redirect('success_page')
        # Return form with errors for HTMX
        return render(request, 'partials/form.html', {'form': form})
```

### Real-time Updates

```html
<div hx-get="/api/live-data" 
     hx-target="#live-content" 
     hx-trigger="every 5s"
     x-data="{ lastUpdate: new Date() }"
     @htmx:afterRequest="lastUpdate = new Date()">
    <div id="live-content"></div>
    <small x-text="'Last updated: ' + lastUpdate.toLocaleTimeString()"></small>
</div>
```

This tech stack provides a modern, performant foundation for building interactive Django applications. Focus on leveraging each tool's strengths while maintaining clean separation of concerns.
