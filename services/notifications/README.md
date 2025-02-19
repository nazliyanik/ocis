# Notification

The notification service is responsible for sending emails to users informing them about events that happened. To do this, it hooks into the event system and listens for certain events that the users need to be informed about.

## Email Notification Templates

The `notifications` service has embedded email text and html body templates.

Email templates can use the placeholders `{{ .Greeting }}`, `{{ .MessageBody }}` and `{{ .CallToAction }}` which are replaced with translations when sent, see the [Translations](#translations) section for more details. Though the email subject is also part of translations, it has no placeholder as it is a mandatory email component.

Depending on the email purpose, placeholders will contain different strings. An individual translatable string is available for each purpose, finally resolved by the placeholder. The embedded templates are available for all deployment scenarios.

```text
template
  placeholders
    translated strings <-- source strings <-- purpose
final output
```

In addition, the notifications service supports custom templates. Custom email templates take precedence over the embedded ones. If a custom email template exists, the embedded templates are not used. To configure custom email templates, the `NOTIFICATIONS_EMAIL_TEMPLATE_PATH` environment variable needs to point to a base folder that will contain the email templates and follow the [templates subfolder hierarchy](#templates-subfolder-hierarchy).This path must be available from all instances of the notifications service, a shared storage is recommended.
```text
{NOTIFICATIONS_EMAIL_TEMPLATE_PATH}/templates/text/email.text.tmpl
{NOTIFICATIONS_EMAIL_TEMPLATE_PATH}/templates/html/email.html.tmpl
{NOTIFICATIONS_EMAIL_TEMPLATE_PATH}/templates/html/img/
```
The source templates provided by ocis you can derive from are located in the following base folder [https://github.com/owncloud/ocis/tree/master/services/notifications/pkg/email/templates](https://github.com/owncloud/ocis/tree/master/services/notifications/pkg/email/templates) with subfolders `templates/text` and `templates/html`.

-   [text/email.text.tmpl](https://github.com/owncloud/ocis/blob/master/services/notifications/pkg/email/templates/text/email.text.tmpl)
-   [html/email.html.tmpl](https://github.com/owncloud/ocis/blob/master/services/notifications/pkg/email/templates/html/email.html.tmpl)

### Templates subfolder hierarchy
```text
templates
│
└───html
│   │   email.html.tmpl
│   │
│   └───img
│       │   logo-mail.gif
│
└───text
    │   email.text.tmpl
```

Custom email templates referenced via `NOTIFICATIONS_EMAIL_TEMPLATE_PATH` must also be located in subfolder `templates/text` and `templates/html` and must have the same names as the embedded templates. It is important that the names of these files and folders match the embedded ones.
The `templates/html` subfolder contains a default HTML template provided by ocis. When using a custom HTML template, hosted images can either be linked with standard HTML code like ```<img src="https://raw.githubusercontent.com/owncloud/core/master/core/img/logo-mail.gif" alt="logo-mail"/>``` or embedded as a CID source ```<img src="cid:logo-mail.gif" alt="logo-mail"/>```. In the latter case, image files must be located in the `templates/html/img` subfolder. Supported embedded image types are png, jpeg, and gif.
Consider that embedding images via a CID resource may not be fully supported in all email web clients.

## Translations

The `notifications` service has embedded translations sourced via transifex to provide a basic set of translated languages. These embedded translations are available for all deployment scenarios.

In addition, the service supports custom translations, though it is currently not possible to just add custom translations to embedded ones. If custom translations are configured, the embedded ones are not used. To configure custom translations,
the `NOTIFICATIONS_TRANSLATION_PATH` environment variable needs to point to a base folder that will contain the translation files. This path must be available from all instances of the notifications service, a shared storage is recommended. Translation files must be of type  [.po](https://www.gnu.org/software/gettext/manual/html_node/PO-Files.html#PO-Files) or [.mo](https://www.gnu.org/software/gettext/manual/html_node/Binaries.html). For each language, the filename needs to be `translations.po` (or `translations.mo`) and stored in a folder structure defining the language code. In general the path/name pattern for a translation file needs to be:

```text
{NOTIFICATIONS_TRANSLATION_PATH}/{language-code}/LC_MESSAGES/translations.po
```

The language code pattern is composed of `language[_territory]` where  `language` is the base language and `_territory` is optional and defines a country.

For example, for the language `de`, one needs to place the corresponding translation files to `{NOTIFICATIONS_TRANSLATION_PATH}/de/LC_MESSAGES/translations.po`.

<!-- also see the userlog readme -->

Important: For the time being, the embedded ownCloud Web frontend only supports the main language code but does not handle any territory. When strings are available in the language code `language_territory`, the web frontend does not see it as it only requests `language`. In consequence, any translations made must exist in the requested `language` to avoid a fallback to the default.

### Translation Rules

*   If a requested language code is not available, the service tries to fall back to the base language if available. For example, if the requested language-code `de_DE` is not available, the service tries to fall back to translations in the `de` folder.
*   If the base language `de` is also not available, the service falls back to the system's default English (`en`),
which is the source of the texts provided by the code.

## Default Language

The default language can be defined via the `OCIS_DEFAULT_LANGUAGE` environment variable. See the `settings` service for a detailed description.
