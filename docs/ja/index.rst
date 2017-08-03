===========
Flask-Login
===========
.. currentmodule:: flask_login

Flask-Login は Flask にユーザーセッションマネージャーを提供します。
これは、ログイン、ログアウト、そしてあなたのユーザーのセッションを長期間記憶するなどの一般的なタスクを管理します。

本機能は:

- アクティブユーザーのIDをセッションに保存し、ログイン・ログアウトを簡単にするでしょう。
- ビューをログインした（またはログアウトした）ユーザーに制限するでしょう。
- 一般的にトリッキーな "remember me" 機能を処理するでしょう。
- ユーザーのセッションクッキーが悪意のある第三者に盗まれるのを防ぐのを助けるでしょう。
- もしかしたら、Flask-Principal や他の認証拡張機能と後から統合することができるかもしれません。

しかし:

- 特定のデータベースまたは他のストレージメソッドを強制しません。あなたはユーザーデータの読み込みの全てを管理します。
- ユーザー名とパスワード、OpenID、またはその他の認証方法を使用するように制限するものではありません。
- "ログインしているかどうか"を超えた権限は処理しません。
- ユーザー登録またはアカウントの回復を処理しません。

.. contents::
   :local:
   :backlinks: none


インストール
============
pip で拡張をインストールする::

    $ pip install flask-login

アプリケーションの設定
======================
Flask-Login を使用するアプリケーションの最も重要な部分は、 `LoginManager` クラスです。 これをアプリケーションのために、あなたのコードのどこかに作成する必要があります。このように::

    login_manager = LoginManager()

ログインマネージャには、アプリケーションと Flask-Login を連携させるためのコードが含まれています。たとえば、ID からユーザーをロードする方法や、ユーザーがログインする必要があるときに送られる場所、そういった類の機能です。

実際のアプリケーションオブジェクトが作成されたら、次のようにログイン用に設定できます::

    login_manager.init_app(app)


使い方
======
`~LoginManager.user_loader` コールバックを提供する必要があります。
このコールバックは、セッションに格納されているユーザーIDからユーザーオブジェクトをリロードするために使用されます。
これはユーザの `unicode` IDを引数に取り、対応するユーザオブジェクトを返す必要があります。
例えば::

    @login_manager.user_loader
    def load_user(user_id):
        return User.get(user_id)

ID が有効でない場合、 `None` を返します（**例外は発生しません**）。
（この場合、ID はセッションから手動で削除され、処理は続行されます）。

ユーザークラス
==============
ユーザーを表すために使用するクラスは、これらのプロパティとメソッドを実装する必要があります:

`is_authenticated`
    このプロパティは、ユーザが認証されている場合、つまり有効な資格情報を持っている場合に `True` を返す必要があります。
    （認証されたユーザだけが `login_required` の基準を満たします。）

`is_active`
    This property should return `True` if this is an active user - in addition to being authenticated, they also have activated their account, not been suspended, or any condition your application has for rejecting an account.
    Inactive accounts may not log in (without being forced of course).

    Fix this translation::

        このプロパティは、アクティブなユーザーであれば `True` を返さなければなりません -
        認証されているだけでなく、アカウントが有効で、資格が停止れていない、またはいかなる状態、アプリケーションがもつ、アカウントを拒否するための。
        非アクティブなアカウントはログインできません（もちろん強制されることはありません）。

`is_anonymous`
    匿名ユーザーの場合、このプロパティは `True` を返す必要があります。
    （実際のユーザーは代わりに `False` を返す必要があります）。

`get_id()`
    このメソッドは、このユーザーを一意に識別する `unicode` を返し、
    `~LoginManager.user_loader` コールバックからユーザーをロードするために使用できなければなりません。
    これは `unicode` **でなければならない** ことに注意してください -
    ID がネイティブの `int` またはその他のタイプの場合は、 `unicode` に変換する必要があります。

ユーザークラスの実装を容易にするために、 `UserMixin` を継承することができます。これは、これらすべてのプロパティとメソッドのデフォルトの実装を提供します。
（ただし、必須ではありません。）

ログインの例
============

ユーザーが認証されたなら、 `login_user` 関数でユーザーをログインさせます。

    例:

.. code-block:: python

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        # ここでは、クライアントサイドのフォームデータを表現して検証するために、ある種のクラスを使用します。
        # たとえば、WTForms はこれを処理するライブラリで、カスタム LoginForm を使用して検証します。
        form = LoginForm()
        if form.validate_on_submit():
            # ログインとユーザの認証。
            # user は `User` クラスのインスタンスでなければいけません。
            login_user(user)

            flask.flash('Logged in successfully.')

            next = flask.request.args.get('next')
            # is_safe_url は url がリダイレクトに対し安全であるかを確認すべきです。
            # 参考までに http://flask.pocoo.org/snippets/62/
            if not is_safe_url(next):
                return flask.abort(400)

            return flask.redirect(next or flask.url_for('index'))
        return flask.render_template('login.html', form=form)

*警告:* あなたは `next` パラメータの値を検証しなければなりません。
そうしないと、アプリケーションはオープンリダイレクトに対して脆弱になります。
`is_safe_url` の実装例については `この Flask Snippet`_ を参照してください。

`current_user` プロキシを使用して、簡単にログインしているユーザにアクセスできます。これは、すべてのテンプレートで利用可能です::

    {% if current_user.is_authenticated %}
      Hi {{ current_user.name }}!
    {% endif %}

ユーザがログインしている必要があるビューは `login_required` デコレータで装飾することができます::

    @app.route("/settings")
    @login_required
    def settings():
        pass

ユーザーがログアウトする準備ができたら::

    @app.route("/logout")
    @login_required
    def logout():
        logout_user()
        return redirect(somewhere)

ログアウトした後、セッションで使われたクッキーはクリーンアップされます。



ログインプロセスのカスタマイズ
==============================
By default, when a user attempts to access a `login_required` view without
being logged in, Flask-Login will flash a message and redirect them to the
log in view. (If the login view is not set, it will abort with a 401 error.)

The name of the log in view can be set as `LoginManager.login_view`.
For example::

    login_manager.login_view = "users.login"

The default message flashed is ``Please log in to access this page.`` To
customize the message, set `LoginManager.login_message`::

    login_manager.login_message = u"Bonvolu ensaluti por uzi tiun paĝon."

To customize the message category, set `LoginManager.login_message_category`::

    login_manager.login_message_category = "info"

When the log in view is redirected to, it will have a ``next`` variable in the
query string, which is the page that the user was trying to access. Alternatively,
if `USE_SESSION_FOR_NEXT` is `True`, the page is stored in the session under the
key ``next``.

If you would like to customize the process further, decorate a function with
`LoginManager.unauthorized_handler`::

    @login_manager.unauthorized_handler
    def unauthorized():
        # do stuff
        return a_response


Login using Authorization header
================================

.. Caution::
   This method will be deprecated; use the `~LoginManager.request_loader`
   below instead.

Sometimes you want to support Basic Auth login using the `Authorization`
header, such as for api requests. To support login via header you will need
to provide a `~LoginManager.header_loader` callback. This callback should behave
the same as your `~LoginManager.user_loader` callback, except that it accepts
a header value instead of a user id. For example::

    @login_manager.header_loader
    def load_user_from_header(header_val):
        header_val = header_val.replace('Basic ', '', 1)
        try:
            header_val = base64.b64decode(header_val)
        except TypeError:
            pass
        return User.query.filter_by(api_key=header_val).first()

By default the `Authorization` header's value is passed to your
`~LoginManager.header_loader` callback. You can change the header used with
the `AUTH_HEADER_NAME` configuration.


Custom Login using Request Loader
=================================
Sometimes you want to login users without using cookies, such as using header
values or an api key passed as a query argument. In these cases, you should use
the `~LoginManager.request_loader` callback. This callback should behave the
same as your `~LoginManager.user_loader` callback, except that it accepts the
Flask request instead of a user_id.

For example, to support login from both a url argument and from Basic Auth
using the `Authorization` header::

    @login_manager.request_loader
    def load_user_from_request(request):

        # first, try to login using the api_key url arg
        api_key = request.args.get('api_key')
        if api_key:
            user = User.query.filter_by(api_key=api_key).first()
            if user:
                return user

        # next, try to login using Basic Auth
        api_key = request.headers.get('Authorization')
        if api_key:
            api_key = api_key.replace('Basic ', '', 1)
            try:
                api_key = base64.b64decode(api_key)
            except TypeError:
                pass
            user = User.query.filter_by(api_key=api_key).first()
            if user:
                return user

        # finally, return None if both methods did not login the user
        return None


Anonymous Users
===============
By default, when a user is not actually logged in, `current_user` is set to
an `AnonymousUserMixin` object. It has the following properties and methods:

- `is_active` and `is_authenticated` are `False`
- `is_anonymous` is `True`
- `get_id()` returns `None`

If you have custom requirements for anonymous users (for example, they need
to have a permissions field), you can provide a callable (either a class or
factory function) that creates anonymous users to the `LoginManager` with::

    login_manager.anonymous_user = MyAnonymousUser


Remember Me
===========
By default, when the user closes their browser the Flask Session is deleted
and the user is logged out. "Remember Me" prevents the user from accidentally
being logged out when they close their browser. This does **NOT** mean
remembering or pre-filling the user's username or password in a login form
after the user has logged out.

"Remember Me" functionality can be tricky to implement. However, Flask-Login
makes it nearly transparent - just pass ``remember=True`` to the `login_user`
call. A cookie will be saved on the user's computer, and then Flask-Login
will automatically restore the user ID from that cookie if it is not in the
session. The cookie is tamper-proof, so if the user tampers with it (i.e.
inserts someone else's user ID in place of their own), the cookie will merely
be rejected, as if it was not there.

That level of functionality is handled automatically. However, you can (and
should, if your application handles any kind of sensitive data) provide
additional infrastructure to increase the security of your remember cookies.


Alternative Tokens
==================
Using the user ID as the value of the remember token means you must change the
user's ID to invalidate their login sessions. One way to improve this is to use
an alternative session token instead of the user's ID. For example::

    @login_manager.user_loader
    def load_user(session_token):
        return User.query.filter_by(session_token=session_token).first()

Then the `~UserMixin.get_id` method of your User class would return the session
token instead of the user's ID::

    def get_id(self):
        return unicode(self.session_token)

This way you are free to change the user's session token to a new randomly
generated value when the user changes their password, which would ensure their
old authentication sessions will cease to be valid. Note that the session
token must still uniquely identify the user... think of it as a second user ID.


Fresh Logins
============
When a user logs in, their session is marked as "fresh," which indicates that
they actually authenticated on that session. When their session is destroyed
and they are logged back in with a "remember me" cookie, it is marked as
"non-fresh." `login_required` does not differentiate between freshness, which
is fine for most pages. However, sensitive actions like changing one's
personal information should require a fresh login. (Actions like changing
one's password should always require a password re-entry regardless.)

`fresh_login_required`, in addition to verifying that the user is logged
in, will also ensure that their login is fresh. If not, it will send them to
a page where they can re-enter their credentials. You can customize its
behavior in the same ways as you can customize `login_required`, by setting
`LoginManager.refresh_view`, `~LoginManager.needs_refresh_message`, and
`~LoginManager.needs_refresh_message_category`::

    login_manager.refresh_view = "accounts.reauthenticate"
    login_manager.needs_refresh_message = (
        u"To protect your account, please reauthenticate to access this page."
    )
    login_manager.needs_refresh_message_category = "info"

Or by providing your own callback to handle refreshing::

    @login_manager.needs_refresh_handler
    def refresh():
        # do stuff
        return a_response

To mark a session as fresh again, call the `confirm_login` function.


Cookie Settings
===============
The details of the cookie can be customized in the application settings.

=========================== =================================================
`REMEMBER_COOKIE_NAME`      The name of the cookie to store the "remember me"
                            information in. **Default:** ``remember_token``
`REMEMBER_COOKIE_DURATION`  The amount of time before the cookie expires, as
                            a `datetime.timedelta` object.
                            **Default:** 365 days (1 non-leap Gregorian year)
`REMEMBER_COOKIE_DOMAIN`    If the "Remember Me" cookie should cross domains,
                            set the domain value here (i.e. ``.example.com``
                            would allow the cookie to be used on all
                            subdomains of ``example.com``).
                            **Default:** `None`
`REMEMBER_COOKIE_PATH`      Limits the "Remember Me" cookie to a certain path.
                            **Default:** ``/``
`REMEMBER_COOKIE_SECURE`    Restricts the "Remember Me" cookie's scope to
                            secure channels (typically HTTPS).
                            **Default:** `None`
`REMEMBER_COOKIE_HTTPONLY`  Prevents the "Remember Me" cookie from being
                            accessed by client-side scripts.
                            **Default:** `False`
=========================== =================================================


Session Protection
==================
While the features above help secure your "Remember Me" token from cookie
thieves, the session cookie is still vulnerable. Flask-Login includes session
protection to help prevent your users' sessions from being stolen.

You can configure session protection on the `LoginManager`, and in the app's
configuration. If it is enabled, it can operate in either `basic` or `strong`
mode. To set it on the `LoginManager`, set the
`~LoginManager.session_protection` attribute to ``"basic"`` or ``"strong"``::

    login_manager.session_protection = "strong"

Or, to disable it::

    login_manager.session_protection = None

By default, it is activated in ``"basic"`` mode. It can be disabled in the
app's configuration by setting the `SESSION_PROTECTION` setting to `None`,
``"basic"``, or ``"strong"``.

When session protection is active, each request, it generates an identifier
for the user's computer (basically, a secure hash of the IP address and user
agent). If the session does not have an associated identifier, the one
generated will be stored. If it has an identifier, and it matches the one
generated, then the request is OK.

If the identifiers do not match in `basic` mode, or when the session is
permanent, then the session will simply be marked as non-fresh, and anything
requiring a fresh login will force the user to re-authenticate. (Of course,
you must be already using fresh logins where appropriate for this to have an
effect.)

If the identifiers do not match in `strong` mode for a non-permanent session,
then the entire session (as well as the remember token if it exists) is
deleted.


Disabling Session Cookie for APIs
=================================
When authenticating to APIs, you might want to disable setting the Flask
Session cookie. To do this, use a custom session interface that skips saving
the session depending on a flag you set on the request. For example::

    from flask import g
    from flask.sessions import SecureCookieSessionInterface
    from flask_login import user_loaded_from_header

    class CustomSessionInterface(SecureCookieSessionInterface):
        """Prevent creating session from API requests."""
        def save_session(self, *args, **kwargs):
            if g.get('login_via_header'):
                return
            return super(CustomSessionInterface, self).save_session(*args,
                                                                    **kwargs)

    app.session_interface = CustomSessionInterface()

    @user_loaded_from_header.connect
    def user_loaded_from_header(self, user=None):
        g.login_via_header = True

This prevents setting the Flask Session cookie whenever the user authenticated
using your `~LoginManager.header_loader`.


Localization
============
By default, the `LoginManager` uses ``flash`` to display messages when a user
is required to log in. These messages are in English. If you require
localization, set the `localize_callback` attribute of `LoginManager` to a
function to be called with these messages before they're sent to ``flash``,
e.g. ``gettext``. This function will be called with the message and its return
value will be sent to ``flash`` instead.


API Documentation
=================
This documentation is automatically generated from Flask-Login's source code.


Configuring Login
-----------------

.. module:: flask_login

.. autoclass:: LoginManager

   .. automethod:: setup_app

   .. automethod:: unauthorized

   .. automethod:: needs_refresh

   .. rubric:: General Configuration

   .. automethod:: user_loader

   .. automethod:: header_loader

   .. attribute:: anonymous_user

      A class or factory function that produces an anonymous user, which
      is used when no one is logged in.

   .. rubric:: `unauthorized` Configuration

   .. attribute:: login_view

      The name of the view to redirect to when the user needs to log in. (This
      can be an absolute URL as well, if your authentication machinery is
      external to your application.)

   .. attribute:: login_message

      The message to flash when a user is redirected to the login page.

   .. automethod:: unauthorized_handler

   .. rubric:: `needs_refresh` Configuration

   .. attribute:: refresh_view

      The name of the view to redirect to when the user needs to
      reauthenticate.

   .. attribute:: needs_refresh_message

      The message to flash when a user is redirected to the reauthentication
      page.

   .. automethod:: needs_refresh_handler


Login Mechanisms
----------------
.. data:: current_user

   A proxy for the current user.

.. autofunction:: login_fresh

.. autofunction:: login_user

.. autofunction:: logout_user

.. autofunction:: confirm_login


Protecting Views
----------------
.. autofunction:: login_required

.. autofunction:: fresh_login_required


User Object Helpers
-------------------
.. autoclass:: UserMixin
   :members:

.. autoclass:: AnonymousUserMixin
   :members:


Utilities
---------
.. autofunction:: login_url


Signals
-------
See the `Flask documentation on signals`_ for information on how to use these
signals in your code.

.. data:: user_logged_in

   Sent when a user is logged in. In addition to the app (which is the
   sender), it is passed `user`, which is the user being logged in.

.. data:: user_logged_out

   Sent when a user is logged out. In addition to the app (which is the
   sender), it is passed `user`, which is the user being logged out.

.. data:: user_login_confirmed

   Sent when a user's login is confirmed, marking it as fresh. (It is not
   called for a normal login.)
   It receives no additional arguments besides the app.

.. data:: user_unauthorized

   Sent when the `unauthorized` method is called on a `LoginManager`. It
   receives no additional arguments besides the app.

.. data:: user_needs_refresh

   Sent when the `needs_refresh` method is called on a `LoginManager`. It
   receives no additional arguments besides the app.

.. data:: session_protected

   Sent whenever session protection takes effect, and a session is either
   marked non-fresh or deleted. It receives no additional arguments besides
   the app.

.. _Flask documentation on signals: http://flask.pocoo.org/docs/signals/
.. _この Flask Snippet: http://flask.pocoo.org/snippets/62/
