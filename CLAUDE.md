# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PyBBM (pybbm) is a Django forum application that provides a full-featured bulletin board system. The project emphasizes easy integration into existing Django sites by focusing on forum-specific functionality rather than reimplementing user management, authentication, or password recovery.

## Repository Structure

```
pyForum/  (PyBBM - Django Forum Application)
│
├── 📄 Core Files
│   ├── setup.py              # Package configuration & dependencies
│   ├── runtests.py           # Test runner entry point
│   ├── tox.ini               # Multi-version testing config
│   ├── README.rst            # Project documentation
│   ├── CLAUDE.md             # AI assistant guidance
│   └── .travis.yml           # CI/CD configuration
│
├── 📦 pybb/                  # Main Django application
│   │
│   ├── Core Application Files
│   │   ├── models.py         # Category, Forum, Topic, Post, Profile, Polls
│   │   ├── views.py          # CBVs for forum display & interactions
│   │   ├── urls.py           # URL routing (ID-based & nice URLs)
│   │   ├── forms.py          # Post, Poll, Subscription forms
│   │   ├── admin.py          # Django admin configuration
│   │   └── apps.py           # Django app configuration
│   │
│   ├── Permission & Access Control
│   │   └── permissions.py    # Extensible permission handler system
│   │
│   ├── Markup System
│   │   └── markup/
│   │       ├── base.py       # Abstract BaseParser class
│   │       ├── bbcode.py     # BBCode markup parser
│   │       └── markdown.py   # Markdown markup parser
│   │
│   ├── Django Integration
│   │   ├── middleware.py     # PybbMiddleware (anonymous tracking)
│   │   ├── context_processors.py  # Template context variables
│   │   ├── compat.py         # Django version compatibility layer
│   │   ├── defaults.py       # All PYBB_* settings with defaults
│   │   └── signals.py        # Django signal handlers
│   │
│   ├── Features
│   │   ├── profiles.py       # PybbProfile base class
│   │   ├── subscription.py   # Topic/Forum subscription logic
│   │   ├── feeds.py          # RSS feeds (posts, topics)
│   │   └── util.py           # Helper functions
│   │
│   ├── 🗄️ management/       # Django management commands
│   │   └── commands/
│   │       ├── pybb_update_counters.py      # Fix forum/topic counters
│   │       ├── pybb_delete_invalid_topics.py # Cleanup orphaned topics
│   │       ├── supermoderator.py            # Grant moderator privileges
│   │       ├── migrate_profile.py           # Profile data migration
│   │       └── dump_topics.py               # Export topics
│   │
│   ├── 🎨 templates/         # Django templates
│   │   └── pybb/             # Forum UI templates
│   │
│   ├── 🎨 static/            # CSS, JS, images
│   │   ├── pybb/             # Forum assets
│   │   └── markitup/         # Markup editor assets
│   │
│   ├── 🌍 locale/            # i18n translations (14 languages)
│   │   ├── de/, es/, fr/, he/, it/, ja/, pl/
│   │   ├── pt_BR/, ru/, sk/, sv/, uk/, zh/
│   │   └── [LC_MESSAGES/django.po]
│   │
│   ├── 📊 migrations/        # Database migrations
│   │   └── 0001_initial.py → 0007_auto_*.py
│   │
│   ├── 🏷️ templatetags/      # Custom template tags
│   │   └── pybb_tags.py      # Forum-specific template helpers
│   │
│   └── tests.py              # Test suite (94%+ coverage)
│
├── 📚 docs/                  # Sphinx documentation
│   ├── index.rst
│   ├── install.rst           # Installation guide
│   ├── settings.rst          # Configuration reference
│   ├── permissions.rst       # Permission system docs
│   ├── customuser.rst        # Custom user model guide
│   ├── markup.rst            # Markup system docs
│   └── [other .rst files]
│
└── 🧪 test/                  # Test projects & examples
    │
    ├── 📦 DEMO PROJECTS (Out-of-the-box examples)
    │
    ├── example_bootstrap/    # Demo #1: Bootstrap-themed forum
    │   ├── manage.py         # Ready to run with ./manage.py runserver
    │   ├── example_bootstrap/
    │   │   ├── settings.py
    │   │   └── urls.py
    │   ├── templates/
    │   │   ├── pybb/         # Forum templates
    │   │   └── registration/ # Auth templates
    │   ├── static/bootstrap/
    │   ├── fixtures/         # Sample data for quick setup
    │   └── requirements.txt  # Uses django-registration-redux, BBCode
    │
    ├── example_thirdparty/   # Demo #2: Third-party integration showcase
    │   ├── manage.py         # Demonstrates pinax-theme, user-accounts
    │   ├── example_thirdparty/
    │   │   ├── settings.py
    │   │   ├── urls.py
    │   │   └── forms.py      # Custom form overrides
    │   ├── templates/pybb/
    │   └── requirements.txt  # Uses postmarkup, captcha, sorl-thumbnail
    │
    └── 🧪 TEST HARNESS

        └── test_project/     # Test suite runner (not a demo)
            ├── manage.py
            ├── test_project/
            │   ├── settings.py   # Test configuration
            │   ├── urls.py
            │   └── markup_parsers.py
            ├── test_app/         # Custom test app
            │   ├── models.py     # Custom user model
            │   └── migrations/
            ├── templates/pybb/   # Template overrides
            └── requirements_test.txt


📋 Key Architecture Flow:
═══════════════════════════

  User Request
       ↓
  urls.py → views.py → permissions.py → models.py
       ↓         ↓            ↓              ↓
  templates  forms.py    Profile      Database
       ↓         ↓                         ↓
  context   markup/                  Counters
processors  parsers                  Tracking


🔑 Data Model Hierarchy:
════════════════════════

  Category (name, hidden, slug)
      ↓
  Forum (name, moderators, parent, counters)
      ↓
  Topic (name, sticky, closed, poll_type)
      ↓
  Post (body → body_html, on_moderation)


⚙️ Core Systems:
════════════════

• Permissions: DefaultPermissionHandler (may_*, filter_*)
• Markup: BBCode/Markdown → HTML rendering
• Subscriptions: Topic & Forum subscriptions with notifications
• Moderation: Pre-moderation, moderator actions
• Read Tracking: Per-user topic/forum read status
• Polls: Single/multiple choice voting
```

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

### Running Demo Projects Locally

The repository includes two out-of-the-box demo projects that can be run locally.

**Important Compatibility Note:** The demo projects' requirements.txt files specify Django 1.8-1.11, but modern Python (3.8+) requires Django 4.2+. You'll need to update the Django version before installation.

#### Demo #1 - Bootstrap Theme (Recommended for Getting Started)

**Location:** `test/example_bootstrap/`

**What you get:**
- Simple Bootstrap UI
- User registration/authentication via django-registration-redux
- BBCode markup parser
- Sample forum data with pre-created users

**Setup Steps:**

```bash
cd test/example_bootstrap

# 1. Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 2. Install pybb in development mode from repo root
pip install -e ../../

# 3. Update requirements.txt first
#    Change: django>=1.8,<1.11
#    To:     django>=4.2,<5.0
#    Then install dependencies:
pip install -r requirements.txt

# 4. Add ALLOWED_HOSTS to example_bootstrap/settings.py
#    Add this line after SECRET_KEY:
#    ALLOWED_HOSTS = ['localhost', '127.0.0.1']

# 5. Create database and run migrations
python manage.py migrate

# 6. Load demo data (includes pre-created users)
python manage.py loaddata fixtures/demo_data.json

# 7. Start development server
python manage.py runserver
```

**Access the forum:**
- URL: http://localhost:8000/forum/
- Admin user: `admin` / `admin` (superuser)
- Moderator user: `moderator` / `moderator`
- Register new users at: http://localhost:8000/accounts/register/

#### Demo #2 - Third-Party Integration (Advanced)

**Location:** `test/example_thirdparty/`

**What you get:**
- pinax-theme-bootstrap theming
- django-user-accounts for authentication
- django-simple-captcha integration
- postmarkup as alternative markup parser
- sorl-thumbnail for image handling
- django-pure-pagination

**Setup Steps:**

```bash
cd test/example_thirdparty

# 1. Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 2. Install pybb in development mode from repo root
pip install -e ../../

# 3. Update requirements.txt first
#    Change: django>=1.8,<1.11
#    To:     django>=4.2,<5.0
#    Then install dependencies:
pip install -r requirements.txt

# 4. Add ALLOWED_HOSTS to example_thirdparty/settings.py
#    Add this line after SECRET_KEY:
#    ALLOWED_HOSTS = ['localhost', '127.0.0.1']

# 5. Create database and run migrations
python manage.py migrate

# 6. Create a superuser account
python manage.py createsuperuser

# 7. Start development server
python manage.py runserver
```

**Access the forum:**
- URL: http://localhost:8000/
- Login with the superuser account you created

**⚠️ Potential Issues:**
- Some third-party packages (pinax-theme-bootstrap, django-user-accounts) may have compatibility issues with Django 4.2
- You may need to update or replace these packages if you encounter errors
- Check for updated versions or Django 4.2-compatible alternatives

#### Troubleshooting Demo Projects

**Issue:** `ImportError` or package compatibility errors
- **Solution:** Try installing updated versions of the problematic packages or check if Django 4.2-compatible versions exist

**Issue:** `DisallowedHost` error when accessing the site
- **Solution:** Ensure you added `ALLOWED_HOSTS = ['localhost', '127.0.0.1']` to the settings.py file

**Issue:** Database errors during migration
- **Solution:** Delete the `testdb.sqlite` file and run `python manage.py migrate` again

**Issue:** Static files not loading
- **Solution:** Run `python manage.py collectstatic` or ensure `DEBUG = True` in settings.py (development only)

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
