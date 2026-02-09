# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PyBBM (pybbm) is a Django forum application that provides a full-featured bulletin board system. The project emphasizes easy integration into existing Django sites by focusing on forum-specific functionality rather than reimplementing user management, authentication, or password recovery.

## Development Commands

### Running Tests

Run the full test suite using the test runner:
```bash
python setup.py test
```

Or run tests directly with the test project's manage.py:
```bash
python test/test_project/manage.py test
```

Run tests for specific app:
```bash
python runtests.py pybb
```

Run with failfast option:
```bash
python runtests.py --failfast
```

### Using Tox

Run tests across multiple Python/Django versions:
```bash
tox
```

Run specific environment:
```bash
tox -e py310-django42-sqlite
```

Run coverage:
```bash
tox -e coverage
```

### Database Migrations

Create migrations:
```bash
python test/test_project/manage.py makemigrations pybb
```

Apply migrations:
```bash
python test/test_project/manage.py migrate pybb
```

### Running Example Projects

The project includes test/example projects:
```bash
cd test/example_bootstrap
python manage.py runserver
```

## Architecture

### Core Models (pybb/models.py)

The forum hierarchy is: **Category → Forum → Topic → Post**

- **Category**: Top-level organizational unit. Has `hidden` field for staff-only visibility.
- **Forum**: Contains topics. Supports nested forums via `parent` field. Has moderators M2M relationship. Tracks counters (post_count, topic_count) and updated timestamp.
- **Topic**: Discussion thread. Has `sticky`, `closed`, `on_moderation` flags. Supports polls via `poll_type`, `poll_question` fields. Uses slug-based URLs when `PYBB_NICE_URL` is enabled.
- **Post**: Individual message in a topic. Inherits from `RenderableItem` which provides `body`, `body_html`, `body_text` fields for markup rendering.
- **Profile**: Default profile model using AutoOneToOneField. Projects can use custom user models or profiles.
- **Read Trackers**: `TopicReadTracker` and `ForumReadTracker` track per-user read status. Custom managers handle MySQL REPEATABLE READ transaction races.
- **Poll Models**: `PollAnswer` and `PollAnswerUser` for poll functionality.
- **ForumSubscription**: Users can subscribe to forums with TYPE_NOTIFY (new topics only) or TYPE_SUBSCRIBE (auto-subscribe to all topics).

### Key Model Behaviors

- **Counter Updates**: Forums and topics maintain denormalized counters. Call `update_counters()` after modifications. Post save/delete automatically updates parent counters.
- **Slug Generation**: `create_or_check_slug()` in models.py generates unique slugs with numeric suffixes when duplicates exist. Limited by `PYBB_NICE_URL_SLUG_DUPLICATE_LIMIT` (default 100).
- **Post Rendering**: Posts use `render()` method to convert markup (BBCode/Markdown) to HTML. The `body_html` field stores rendered HTML, `body_text` stores stripped plain text.
- **Topic Head**: The first post in a topic is the "head". Deleting the head post deletes the entire topic.

### Views Architecture (pybb/views.py)

Views follow class-based view patterns with mixins:

- **PaginatorMixin**: Handles pagination, supports django-pure-pagination if installed.
- **RedirectToLoginMixin**: Redirects unauthenticated users to login on PermissionDenied, returns 403 for authenticated users.
- **Main Views**: IndexView, CategoryView, ForumView, TopicView, PostView use DetailView/ListView patterns.
- **Post Creation**: AddPostView handles both new topics and replies using same view.
- **Moderation**: StickTopicView, CloseTopicView, ModeratePost etc. for moderator actions.

### Permission System (pybb/permissions.py)

PyBBM uses an extensible permission handler pattern:

- **DefaultPermissionHandler**: Base class with `filter_*` (queryset filtering) and `may_*` (boolean checks) methods.
- **Customization**: Override by setting `PYBB_PERMISSION_HANDLER` to your custom class path.
- **Key Concepts**:
  - Superusers have all permissions
  - Staff can see hidden categories/forums (note: `is_staff` checks may need refinement for production)
  - Moderators assigned per-forum via M2M relationship
  - Pre-moderation: Posts can be `on_moderation` until approved by moderators
  - Authors can view their own on_moderation posts

### Markup System (pybb/markup/)

Pluggable markup parsers:

- **base.py**: BaseParser abstract class. Handles smile replacement, body cleaning (rstrip, filter_blanks).
- **bbcode.py**: BBCodeParser using python-bbcode library.
- **markdown.py**: MarkdownParser using python-markdown library.
- **Configuration**: Set `PYBB_MARKUP` to 'bbcode' or 'markdown'. Define custom parsers in `PYBB_MARKUP_ENGINES_PATHS`.
- **Quote Functionality**: Markup parsers implement quote rendering (e.g., BBCode [quote] tags).

### URL Routing (pybb/urls.py)

Two URL schemes:

1. **ID-based (default)**: `/forum/123/`, `/topic/456/`
2. **Nice URLs**: Enabled via `PYBB_NICE_URL` setting. Uses `/c/category-slug/forum-slug/topic-slug/` format.

When PYBB_NICE_URL is enabled, both URL patterns are registered, with ID-based as fallback.

### Settings (pybb/defaults.py)

All settings have `PYBB_` prefix. Key settings:

- **PYBB_MARKUP**: 'bbcode' or 'markdown'
- **PYBB_NICE_URL**: Enable slug-based URLs
- **PYBB_PREMODERATION**: Require moderator approval for posts
- **PYBB_ENABLE_ANONYMOUS_POST**: Allow non-authenticated posting
- **PYBB_PERMISSION_HANDLER**: Path to custom permission handler
- **PYBB_PROFILE_RELATED_NAME**: Related name for profile model (None for custom user model)
- **PYBB_ATTACHMENT_ENABLE**: Enable file attachments
- **PYBB_DISABLE_SUBSCRIPTIONS**: Turn off subscription features
- **PYBB_ALLOW_DELETE_OWN_POST**: Users can delete their own posts

Settings use `getattr(settings, 'SETTING_NAME', default_value)` pattern.

### Management Commands (pybb/management/commands/)

- **pybb_update_counters.py**: Recalculate all forum/topic counters (use after data corruption)
- **pybb_delete_invalid_topics.py**: Remove topics without posts
- **migrate_profile.py**: Migrate profile data
- **supermoderator.py**: Grant user moderator status on all forums
- **dump_topics.py**: Export topics

### Template Tags (pybb/templatetags/pybb_tags.py)

Custom template tags for forum display. Includes read/unread status, pagination helpers, permission checks.

### Compatibility Layer (pybb/compat.py)

Handles differences across Django versions. Key functions:
- `get_user_model()` / `get_user_model_path()`: Django user model helpers
- `get_username_field()`: Username field name
- `get_atomic_func()`: Transaction handling
- `slugify()`: Unicode-aware slug generation

## Testing Strategy

Tests are in `pybb/tests.py`. The test suite achieves 94%+ coverage.

Test project configuration in `test/test_project/test_project/settings.py` provides example integration.

Tox tests against multiple Python (3.8-3.11) and Django (4.2) versions with sqlite/postgres/mysql.

## Integration Notes

When integrating PyBBM:

1. **User Model**: PyBBM works with Django's default User or custom user models. Profile must inherit from `PybbProfile` or provide equivalent fields.
2. **Middleware**: Add `pybb.middleware.PybbMiddleware` to track anonymous user views.
3. **Context Processor**: Add `pybb.context_processors.processor` for template context.
4. **Base Template**: PyBBM extends `PYBB_TEMPLATE` (default: "base.html") which must provide a `content` block.
5. **URLs**: Include pybb.urls with namespace='pybb'.
6. **Permissions**: Users need Django permissions like `pybb.add_post`, `pybb.change_post` for basic forum participation. Set `PYBB_AUTO_USER_PERMISSIONS` to auto-grant on user creation.

## Common Development Patterns

### Adding a New Permission Check

1. Add `may_*` method to custom permission handler (inherit from DefaultPermissionHandler)
2. Call permission check in view: `perms.may_do_something(request.user, object)`
3. Set `PYBB_PERMISSION_HANDLER` to your custom class

### Creating a Custom Markup Parser

1. Subclass `pybb.markup.base.BaseParser`
2. Implement `format(self, text, instance=None)` method
3. Add to `PYBB_MARKUP_ENGINES_PATHS` and set `PYBB_MARKUP`

### Extending Models

PyBBM models aren't abstract, so use these approaches:
- **Profile**: Extend PybbProfile in your own profile model
- **Signals**: Use Django signals (in pybb/signals.py) to hook into post/topic lifecycle
- **Monkey Patching**: Add methods to models (not recommended for production)

### Working with Counters

After bulk operations or data imports, counters may be incorrect:
```python
forum.update_counters()  # Recalculate forum counters
topic.update_counters()  # Recalculate topic counters
```

Or use management command:
```bash
python manage.py pybb_update_counters
```
