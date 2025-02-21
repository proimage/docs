# Upgrading from Craft 2

The first step to upgrading your site to Craft 3 is updating the CMS itself.

## Preparing for the Upgrade

Before you begin, make sure that:

- you’ve reviewed the changes in Craft 3 further down this page
- your server meets Craft 3’s [minimum requirements](requirements.md) (Craft 3 requires PHP 7.2.5+ and at least 256 MB of memory allocated to PHP)
- your site is running at least **Craft 2.6.2788**
- your plugins are all up-to-date, and you’ve verified that they’ve been updated for Craft 3 (you can see a report of your plugins’ Craft 3 compatibility status from the Updates page in the Craft 2 control panel)
- your **database is backed up** in case everything goes horribly wrong

Once you've completed everything listed above you can continue with the upgrade process.

## Performing the Upgrade

The best way to upgrade a Craft 2 site is to approach it like you’re building a new Craft 3 site. So to begin, create a new directory alongside your current project, and follow steps 1-3 in the [installation instructions](installation.md).

With Craft 3 downloaded and prepped, follow these steps to complete the upgrade:

1. Configure the `.env` file in your new project with your database connection settings from your old `craft/config/db.php` file.

   ::: tip
   Don’t forget to set `DB_TABLE_PREFIX="craft"` if that’s what your database tables are prefixed with.
   :::

2. Copy any settings from your old `craft/config/general.php` file into your new project’s `config/general.php` file.

3. Copy your old `craft/config/license.key` file into your new project’s `config/` folder.

4. Copy your old custom Redactor config files from `craft/config/redactor/` over to your new project’s `config/redactor/` directory.

5. Copy your old custom login page logo and site icon files from `craft/storage/rebrand/` over to your new project’s `storage/rebrand/` directory.

6. Copy your old user photos from `craft/storage/userphotos/` over to your new project’s `storage/userphotos/` directory.

7. Copy your old templates from `craft/templates/` over to your new project’s `templates/` directory.

8. If you had made any changes to your `public/index.php` file, copy them to your new project’s `web/index.php` file.

9. Copy any other files in your old `public/` directory into your new project’s `web/` directory.

10. Update your web server to point to your new project’s `web/` directory.

11. Point your browser to your control panel URL (e.g. `http://my-project.test/admin`). If you see the update prompt, you did everything right! Go ahead and click “Finish up” to update your database.

12. If you had any plugins installed, you’ll need to install their Craft 3 counterparts from the “Plugin Store” section in the control panel. (See the plugins’ documentation for any additional upgrade instructions.)

Now that you’ve upgraded your install to use Craft 3, please take some time to review the changes on this page and update your project to follow the changes in Craft 3.

### Troubleshooting

#### I get the Craft installer when I access my control panel.

If this happens, it’s because your database connection settings in the `.env` file don’t quite match up to what they used to be. Most likely you forgot to set the correct `DB_TABLE_PREFIX`.

#### I’m getting a “Setting unknown property: craft\config\DbConfig::initSQLs” error.

The `initSQLs` database config setting was removed in Craft 3, as it was generally only used to fix MySQL 5.7 support in Craft 2, which isn’t necessary in Craft 3. Just open your `config/db.php` file and delete the line that begins with `'initSQLs'`.

## Rich Text Fields

The “Rich Text” field type has been removed from Craft 3, in favor of new [Redactor](https://github.com/craftcms/redactor) and [CKEditor](https://github.com/craftcms/ckeditor) plugins.

If you have any existing Rich Text fields, they will be automatically converted to Redactor fields when you install the Redactor plugin.

### Redactor Configs

If you do install the Redactor plugin, you will need to ensure that your Redactor configs in `config/redactor/` are valid JSON. That means:

- No comments
- All object properties (the config setting names) must be wrapped in double quotes
- All strings must use double quotes rather than single quotes

```javascript
// Bad:
{
  /* interesting comment */
  buttons: ['bold', 'italic']
}

// Good:
{
  "buttons": ["bold", "italic"]
}
```

## Position Select Fields

The “Position Select” field type has been removed from Craft 3. If you had any Position Select fields, they will be converted to Dropdown fields, with all the same options.

If you miss Position Select, you can try installing the [Position Fieldtype](https://github.com/Rias500/craft-position-fieldtype) plugin, which brings it back.

## Lightswitch Fields

Lightswitch field values are now always `true` or `false`. If you’re accessing a Lightswitch field value for an element that doesn’t have an explicit value set yet, the field’s default value will be returned instead.

## Remote Volumes

Support for Amazon S3, Rackspace Cloud Files, and Google Cloud Storage have been moved into plugins. If you have any asset volumes that were using those services in Craft 2, you will need to install the new plugins:

- [Amazon S3](https://github.com/craftcms/aws-s3)
- [Rackspace Cloud Files](https://github.com/craftcms/rackspace)
- [Google Cloud Storage](https://github.com/craftcms/google-cloud)

## Configuration

### Config Settings

Some general config settings have been renamed in Craft 3. The old setting names have been deprecated, but will continue to work until Craft 4.

| Old Setting                  | New Setting
| ---------------------------- | -----------------------------
| `activateAccountFailurePath` | `invalidUserTokenPath`
| `backupDbOnUpdate`           | `backupOnUpdate`<sup>1</sup>
| `defaultFilePermissions`     | `defaultFileMode`<sup>2</sup>
| `defaultFolderPermissions`   | `defaultDirMode`
| `environmentVariables`       | `aliases` <sup>3</sup>
| `restoreDbOnUpdateFailure`   | `restoreOnUpdateFailure`
| `useWriteFileLock`           | `useFileLocks`
| `validationKey`              | `securityKey`<sup>4</sup>

*<sup>1</sup> Performance should no longer be a major factor when setting `backupOnUpdate` to `false`, since backups aren’t generated by PHP anymore.*

*<sup>2</sup> `defaultFileMode` is now `null` by default, meaning it will be determined by the current environment.*

*<sup>3</sup> Settings that supported values defined by your `environmentVariables` config setting in Craft 2 can now be set to system environment variables and aliases in Craft 3. (See [Environmental Configuration](config/#environmental-configuration) to learn more about those.) Site URL and Local volume settings will automatically be converted to the new `@alias/sub/path` syntax when updating to Craft 3.

*<sup>4</sup> `securityKey` is no longer optional. If you haven’t set it yet, set it to the value in `storage/runtime/validation.key` (if the file exists). The auto-generated `validation.key` file fallback will be removed in Craft 4.*

Some config settings have been removed entirely:

| File          | Setting
| ------------- | -----------
| `db.php`      | `collation`
| `db.php`      | `initSQLs`
| `general.php` | `appId`
| `general.php` | `cacheMethod` (see [Configuration → Data Caching Config](config/README.md#data-caching-config))

### `omitScriptNameInUrls` and `usePathInfo`

The `omitScriptNameInUrls` setting can no longer be set to `'auto'`, as it was by default in Craft 2. Which means you will need to explicitly set it to `true` in `config/general.php` if you’ve configured your server to route HTTP requests to `index.php`.

Similarly, the `usePathInfo` setting can no longer be set to `'auto'` either. If your server is configured to support [PATH_INFO](https://craftcms.com/support/enable-path-info), you can set this to `true`. This is only necessary if you can’t set `omitScriptNameInUrls` to `true`, though.

## URL Rules

If you have any [URL rules](config/README.md#url-rules) saved in `config/routes.php`, you will need to update them to Yii 2’s [pattern-route syntax](https://www.yiiframework.com/doc/guide/2.0/en/runtime-routing#url-rules).

- Named parameters in the pattern should be defined using the format (`<ParamName:RegExp>`) rather than as a regular expression subpattern (`(?P<ParamName>RegExp)`).
- Unnamed parameters are no longer allowed (e.g. `([^\/]+)`). They must also be converted to the new named parameter syntax (`<ParamName:RegExp>`).
- Controller action routes should be defined as a string (`'action/path'`) rather than an array with an `action` key (`['action' => 'action/path']`).
- Template routes should be defined as an array with a `template` key (`['template' => 'template/path']`) rather than a string (`'template/path'`).

```php
// Old:
'dashboard' => ['action' => 'dashboard/index'],
'settings/fields/new' => 'settings/fields/_edit',
'settings/fields/edit/(?P<fieldId>\d+)' => 'settings/fields/_edit',
'blog/type/([^\/]+)' => 'blog/_type',

// New:
'dashboard' => 'dashboard/index',
'settings/fields/new' => ['template' => 'settings/fields/_edit'],
'settings/fields/edit/<fieldId:\d+>' => ['template' => 'settings/fields/_edit'],
'blog/type/<type:[^\/]+>' => ['template' => 'blog/_type'],
```

## PHP Constants

Some PHP constants have been deprecated in Craft 3, and will no longer work in Craft 4:

| Old PHP Constant | What to do instead
| ---------------- | ----------------------------------------
| `CRAFT_LOCALE`   | Use the [CRAFT_SITE](config/README.md#craft-site) constant<sup>1</sup>
| `CRAFT_SITE_URL` | Use [environment variables](config/#environmental-configuration)

*<sup>1</sup> Craft 3 doesn’t require each site/locale to have its own `index.php` file anymore, so alternatively you can remove all unnecessary site/locale web roots and subfolders. See the [Localization guide](sites.md) for more info.*

## Static Translation Files

Craft 3 still supports [static message translations](sites.md#static-message-translations), but the directory structure has changed. Now within your `translations/` folder, you should create subdirectories for each locale, and within them, PHP files for each **translation category**.

The acceptable translation categories are:

| Category        | Description
| --------------- | -----------------------------------------
| `app`           | Craft’s translation messages
| `yii`           | Yii’s translation messages
| `site`          | custom site-specific translation messages
| `plugin-handle` | Plugins’ translation messages

In Craft 3, your `translations/` folder might look something like this:

```treeview
translations/
└── de/
    ├── app.php
    └── site.php
```

## User Photos

User photos are stored as assets now. When upgrading to Craft 3, Craft will automatically create a new asset volume called “User Photos” at `storage/userphotos/` (where Craft previously stored all user photos, but without the `<Username>/` subfolders). However this folder is above your web root and inaccessible to HTTP requests, so until you make this volume publicly accessible, user photos will not work on the front end.

Here’s how you can resolve this:

1. Move the `storage/userphotos/` folder somewhere below your web root (e.g. `web/userphotos/`)
2. Go to Settings → Assets → Volumes → User Photos and configure the volume based on the new folder location:
    - Update the File System Path setting to point to the new folder location
    - Enable the “Assets in this volume have public URLs” setting
    - Set the correct URL setting for the folder
    - Save the volume

## Twig 2

Craft 3 uses Twig 2, which has its own breaking changes for templates:

### Macros

Twig 2 requires that you explicitly import macros in each template where you are using them. They are no longer automatically available if a parent template is including them, or even if they were defined in the same template file.

```twig
Old:
{% macro foo %}...{% endmacro %}
{{ _self.foo() }}

New:
{% macro foo %}...{% endmacro %}
{% import _self as macros %}
{{ macros.foo() }}
```

### Undefined Blocks

Twig 1 let you call `block()` even for blocks that didn’t exist:

```twig
{% if block('foo') is not empty %}
  {{ block('foo') }}
{% endif %}
```

Twig 2 will throw an error unless it’s a `defined` test:

```twig
{% if block('foo') is defined %}
  {{ block('foo') }}
{% endif %}
```

## Template Tags

The [{% paginate %}](dev/tags.md#paginate) tag no longer has an `{% endpaginate %}` closing tag, so remove any instances of that.

Some Twig template tags have been deprecated in Craft 3, and will be completely removed in Craft 4:

| Old Tag                         | What to do instead
| ------------------------------- | ---------------------------------------------
| `{% includeCss %}`              | Use the [{% css %}](dev/tags.md#css) tag
| `{% includeHiResCss %}`         | Use the [{% css %}](dev/tags.md#css) tag and write your own media selector
| `{% includeJs %}`               | Use the [{% js %}](dev/tags.md#js) tag
| `{% includeCssFile url %}`      | `{% do view.registerCssFile(url) %}`
| `{% includeJsFile url %}`       | `{% do view.registerJsFile(url) %}`
| `{% includeCssResource path %}` | Use an [asset bundle](extend/asset-bundles.md)
| `{% includeJsResource path %}`  | Use an [asset bundle](extend/asset-bundles.md)

## Template Functions

Some template functions have been removed completely:

| Old Template Function                       | What to do instead
| ------------------------------------------- | ------------------------------------
| `craft.hasPackage()`                        | *(n/a)*
| `craft.entryRevisions.getDraftByOffset()`   | *(n/a)*
| `craft.entryRevisions.getVersionByOffset()` | *(n/a)*
| `craft.fields.getFieldType(type)`           | `craft.app.fields.createField(type)`
| `craft.fields.populateFieldType()`          | *(n/a)*
| `craft.tasks.areTasksPending()`             | `craft.app.queue.getHasWaitingJobs()`<sup>1</sup>
| `craft.tasks.getRunningTask()`              | *(n/a)*
| `craft.tasks.getTotalTasks()`               | *(n/a)*
| `craft.tasks.haveTasksFailed()`             | *(n/a)*
| `craft.tasks.isTaskRunning()`               | `craft.app.queue.getHasReservedJobs()`<sup>1</sup>

*<sup>1</sup> Only available if the `queue` component implements <craft3:craft\queue\QueueInterface>.*

Some template functions have been deprecated in Craft 3, and will be completely removed in Craft 4:

| Old Template Function                                   | What to do instead
| ------------------------------------------------------- | ---------------------------------------------
| `round(num)`                                            | `num|round`
| `getCsrfInput()`                                        | `csrfInput()`
| `getHeadHtml()`                                         | `head()`
| `getFootHtml()`                                         | `endBody()`
| `getTranslations()`                                     | `view.getTranslations()|json_encode|raw`
| `craft.categoryGroups.getAllGroupIds()`                 | `craft.app.categories.allGroupIds`
| `craft.categoryGroups.getEditableGroupIds()`            | `craft.app.categories.editableGroupIds`
| `craft.categoryGroups.getAllGroups()`                   | `craft.app.categories.allGroups`
| `craft.categoryGroups.getEditableGroups()`              | `craft.app.categories.editableGroups`
| `craft.categoryGroups.getTotalGroups()`                 | `craft.app.categories.totalGroups`
| `craft.categoryGroups.getGroupById(id)`                 | `craft.app.categories.getGroupById(id)`
| `craft.categoryGroups.getGroupByHandle(handle)`         | `craft.app.categories.getGroupByHandle(handle)`
| `craft.config.[setting]` *(magic getter)*               | `craft.app.config.general.[setting]`
| `craft.config.get(setting)`                             | `craft.app.config.general.[setting]`
| `craft.config.usePathInfo()`                            | `craft.app.config.general.usePathInfo`
| `craft.config.omitScriptNameInUrls()`                   | `craft.app.config.general.omitScriptNameInUrls`
| `craft.config.getResourceTrigger()`                     | `craft.app.config.general.resourceTrigger`
| `craft.locale()`                                        | `craft.app.language`
| `craft.isLocalized()`                                   | `craft.app.isMultiSite`
| `craft.deprecator.getTotalLogs()`                       | `craft.app.deprecator.totalLogs`
| `craft.elementIndexes.getSources()`                     | `craft.app.elementIndexes.sources`
| `craft.emailMessages.getAllMessages()`                  | `craft.emailMessages.allMessages`
| `craft.emailMessages.getMessage(key)`                   | `craft.app.emailMessages.getMessage(key)`
| `craft.entryRevisions.getDraftsByEntryId(id)`           | `craft.app.entryRevisions.getDraftsByEntryId(id)`
| `craft.entryRevisions.getEditableDraftsByEntryId(id)`   | `craft.entryRevisions.getEditableDraftsByEntryId(id)`
| `craft.entryRevisions.getDraftById(id)`                 | `craft.app.entryRevisions.getDraftById(id)`
| `craft.entryRevisions.getVersionsByEntryId(id)`         | `craft.app.entryRevisions.getVersionsByEntryId(id)`
| `craft.entryRevisions.getVersionById(id)`               | `craft.app.entryRevisions.getVersionById(id)`
| `craft.feeds.getFeedItems(url)`                         | `craft.app.feeds.getFeedItems(url)`
| `craft.fields.getAllGroups()`                           | `craft.app.fields.allGroups`
| `craft.fields.getGroupById(id)`                         | `craft.app.fields.getGroupById(id)`
| `craft.fields.getFieldById(id)`                         | `craft.app.fields.getFieldById(id)`
| `craft.fields.getFieldByHandle(handle)`                 | `craft.app.fields.getFieldByHandle(handle)`
| `craft.fields.getAllFields()`                           | `craft.app.fields.allFields`
| `craft.fields.getFieldsByGroupId(id)`                   | `craft.app.fields.getFieldsByGroupId(id)`
| `craft.fields.getLayoutById(id)`                        | `craft.app.fields.getLayoutById(id)`
| `craft.fields.getLayoutByType(type)`                    | `craft.app.fields.getLayoutByType(type)`
| `craft.fields.getAllFieldTypes()`                       | `craft.app.fields.allFieldTypes`
| `craft.globals.getAllSets()`                            | `craft.app.globals.allSets`
| `craft.globals.getEditableSets()`                       | `craft.app.globals.editableSets`
| `craft.globals.getTotalSets()`                          | `craft.app.globals.totalSets`
| `craft.globals.getTotalEditableSets()`                  | `craft.app.globals.totalEditableSets`
| `craft.globals.getSetById(id)`                          | `craft.app.globals.getSetById(id)`
| `craft.globals.getSetByHandle(handle)`                  | `craft.app.globals.getSetByHandle(handle)`
| `craft.i18n.getAllLocales()`                            | `craft.app.i18n.allLocales`
| `craft.i18n.getAppLocales()`                            | `craft.app.i18n.appLocales`
| `craft.i18n.getCurrentLocale()`                         | `craft.app.locale`
| `craft.i18n.getLocaleById(id)`                          | `craft.app.i18n.getLocaleById(id)`
| `craft.i18n.getSiteLocales()`                           | `craft.app.i18n.siteLocales`
| `craft.i18n.getSiteLocaleIds()`                         | `craft.app.i18n.siteLocaleIds`
| `craft.i18n.getPrimarySiteLocale()`                     | `craft.app.i18n.primarySiteLocale`
| `craft.i18n.getEditableLocales()`                       | `craft.app.i18n.editableLocales`
| `craft.i18n.getEditableLocaleIds()`                     | `craft.app.i18n.editableLocaleIds`
| `craft.i18n.getLocaleData()`                            | `craft.app.i18n.getLocaleById(id)`
| `craft.i18n.getDatepickerJsFormat()`                    | `craft.app.locale.getDateFormat('short', 'jui')`
| `craft.i18n.getTimepickerJsFormat()`                    | `craft.app.locale.getTimeFormat('short', 'php')`
| `craft.request.isGet()`                                 | `craft.app.request.isGet`
| `craft.request.isPost()`                                | `craft.app.request.isPost`
| `craft.request.isDelete()`                              | `craft.app.request.isDelete`
| `craft.request.isPut()`                                 | `craft.app.request.isPut`
| `craft.request.isAjax()`                                | `craft.app.request.isAjax`
| `craft.request.isSecure()`                              | `craft.app.request.isSecureConnection`
| `craft.request.isLivePreview()`                         | `craft.app.request.isLivePreview`<sup>1</sup>
| `craft.request.getScriptName()`                         | `craft.app.request.scriptFilename`
| `craft.request.getPath()`                               | `craft.app.request.pathInfo`
| `craft.request.getUrl()`                                | `url(craft.app.request.pathInfo)`
| `craft.request.getSegments()`                           | `craft.app.request.segments`
| `craft.request.getSegment(num)`                         | `craft.app.request.getSegment(num)`
| `craft.request.getFirstSegment()`                       | `craft.app.request.segments|first`
| `craft.request.getLastSegment()`                        | `craft.app.request.segments|last`
| `craft.request.getParam(name)`                          | `craft.app.request.getParam(name)`
| `craft.request.getQuery(name)`                          | `craft.app.request.getQueryParam(name)`
| `craft.request.getPost(name)`                           | `craft.app.request.getBodyParam(name)`
| `craft.request.getCookie(name)`                         | `craft.app.request.cookies.get(name)`
| `craft.request.getServerName()`                         | `craft.app.request.serverName`
| `craft.request.getUrlFormat()`                          | `craft.app.config.general.usePathInfo`
| `craft.request.isMobileBrowser()`                       | `craft.app.request.isMobileBrowser()`
| `craft.request.getPageNum()`                            | `craft.app.request.pageNum`
| `craft.request.getHostInfo()`                           | `craft.app.request.hostInfo`
| `craft.request.getScriptUrl()`                          | `craft.app.request.scriptUrl`
| `craft.request.getPathInfo()`                           | `craft.app.request.getPathInfo(true)`
| `craft.request.getRequestUri()`                         | `craft.app.request.url`
| `craft.request.getServerPort()`                         | `craft.app.request.serverPort`
| `craft.request.getUrlReferrer()`                        | `craft.app.request.referrer`
| `craft.request.getUserAgent()`                          | `craft.app.request.userAgent`
| `craft.request.getUserHostAddress()`                    | `craft.app.request.userIP`
| `craft.request.getUserHost()`                           | `craft.app.request.userHost`
| `craft.request.getPort()`                               | `craft.app.request.port`
| `craft.request.getCsrfToken()`                          | `craft.app.request.csrfToken`
| `craft.request.getQueryString()`                        | `craft.app.request.queryString`
| `craft.request.getQueryStringWithoutPath()`             | `craft.app.request.queryStringWithoutPath`
| `craft.request.getIpAddress()`                          | `craft.app.request.userIP`
| `craft.request.getClientOs()`                           | `craft.app.request.clientOs`
| `craft.sections.getAllSections()`                       | `craft.app.sections.allSections`
| `craft.sections.getEditableSections()`                  | `craft.app.sections.editableSections`
| `craft.sections.getTotalSections()`                     | `craft.app.sections.totalSections`
| `craft.sections.getTotalEditableSections()`             | `craft.app.sections.totalEditableSections`
| `craft.sections.getSectionById(id)`                     | `craft.app.sections.getSectionById(id)`
| `craft.sections.getSectionByHandle(handle)`             | `craft.app.sections.getSectionByHandle(handle)`
| `craft.systemSettings.[category]` *(magic getter)*      | `craft.app.systemSettings.getSettings('category')`
| `craft.userGroups.getAllGroups()`                       | `craft.app.userGroups.allGroups`
| `craft.userGroups.getGroupById(id)`                     | `craft.app.userGroups.getGroupById(id)`
| `craft.userGroups.getGroupByHandle(handle)`             | `craft.app.userGroups.getGroupByHandle(handle)`
| `craft.userPermissions.getAllPermissions()`             | `craft.app.userPermissions.allPermissions`
| `craft.userPermissions.getGroupPermissionsByUserId(id)` | `craft.app.userPermissions.getGroupPermissionsByUserId(id)`
| `craft.session.isLoggedIn()`                            | `not craft.app.user.isGuest`
| `craft.session.getUser()`                               | `currentUser`
| `craft.session.getRemainingSessionTime()`               | `craft.app.user.remainingSessionTime`
| `craft.session.getRememberedUsername()`                 | `craft.app.user.rememberedUsername`
| `craft.session.getReturnUrl()`                          | `craft.app.user.getReturnUrl()`
| `craft.session.getFlashes()`                            | `craft.app.session.getAllFlashes()`
| `craft.session.getFlash()`                              | `craft.app.session.getFlash()`
| `craft.session.hasFlash()`                              | `craft.app.session.hasFlash()`

*<sup>1</sup> `craft.app.request.isLivePreview` is also deprecated, and only will return `true` when previewing categories or plugin-supplied element types that don’t support the new previewing system. If you were calling this to work around Craft templating bugs in Live Preview requests, you can simply delete the condition now, and treat Live Preview requests the same as any other request type.*

## Date Formatting

Craft’s extended DateTime class has been removed in Craft 3. Here’s a list of things you used to be able to do in your templates, and what the Craft 3 equivalent is. (The DateTime object is represented by the `d` variable. In reality it could be `entry.postDate`, `now`, etc.)

| Old                               | New
| --------------------------------- | ----------------------------------
| `{{ d }}` *(treated as a string)* | `{{ d|date('Y-m-d') }}`
| `{{ d.atom() }}`                  | `{{ d|atom }}`
| `{{ d.cookie() }}`                | `{{ d|date('l, d-M-y H:i:s T')}}`
| `{{ d.day() }}`                   | `{{ d|date('j') }}`
| `{{ d.iso8601() }}`               | `{{ d|date('c') }}`
| `{{ d.localeDate() }}`            | `{{ d|date('short') }}`
| `{{ d.localeTime() }}`            | `{{ d|time('short') }}`
| `{{ d.month() }}`                 | `{{ d|date('n') }}`
| `{{ d.mySqlDateTime() }}`         | `{{ d|date('Y-m-d H:i:s') }}`
| `{{ d.nice() }}`                  | `{{ d|datetime('short') }}`
| `{{ d.rfc1036() }}`               | `{{ d|date('D, d M y H:i:s O') }}`
| `{{ d.rfc1123() }}`               | `{{ d|date('r') }}`
| `{{ d.rfc2822() }}`               | `{{ d|date('r') }}`
| `{{ d.rfc3339() }}`               | `{{ d|date('Y-m-d\\TH:i:sP') }}`
| `{{ d.rfc822() }}`                | `{{ d|date('D, d M y H:i:s O') }}`
| `{{ d.rfc850() }}`                | `{{ d|date('l, d-M-y H:i:s T') }}`
| `{{ d.rss() }}`                   | `{{ d|rss }}`
| `{{ d.uiTimestamp() }}`           | `{{ d|timestamp('short') }}`
| `{{ d.w3c() }}`                   | `{{ d|date('Y-m-d\\TH:i:sP') }}`
| `{{ d.w3cDate() }}`               | `{{ d|date('Y-m-d') }}`
| `{{ d.year() }}`                  | `{{ d|date('Y') }}`

## Currency Formatting

The `|currency` filter now maps to <craft3:craft\i18n\Formatter::asCurrency()>. It still works the same, but the `stripZeroCents` argument has been renamed to `stripZeros`, and pushed back a couple notches, so you will need to update your templates if you were setting that argument.

```twig
Old:
{{ num|currency('USD', true) }}
{{ num|currency('USD', stripZeroCents = true) }}

New:
{{ num|currency('USD', stripZeros = true) }}
```

## Element Queries

### Query Params

Some element query params have been removed:

| Element Type | Old Param          | What to do instead
| ------------ | ------------------ | -------------------------
| All of them  | `childOf`          | Use a `relatedTo` param with a `sourceElement` key
| All of them  | `childField`       | Use a `relatedTo` param with a `field` key
| All of them  | `parentOf`         | Use a `relatedTo` param with a `targetElement` key
| All of them  | `parentField`      | Use a `relatedTo` param with a `field` key
| All of them  | `depth`            | Use the `level` param
| Tag          | `name`             | Use the `title` param
| Tag          | `setId`            | Use the `groupId` param
| Tag          | `set`              | Use the `group` param
| Tag          | `orderBy:"name"`   | Set the `orderBy` param to `'title'`

Some element query params have been renamed in Craft 3. The old params have been deprecated, but will continue to work until Craft 4.

| Element Type | Old Param                | New Param
| ------------ | ------------------------ | ----------------------------
| All of them  | `order`                  | `orderBy`
| All of them  | `locale`                 | `siteId` or `site`
| All of them  | `localeEnabled`          | `enabledForSite`
| All of them  | `relatedTo.sourceLocale` | `relatedTo.sourceSite`
| Asset        | `source`                 | `volume`
| Asset        | `sourceId`               | `volumeId`
| Matrix Block | `ownerLocale`            | `site` or `siteId`

#### `limit` Param

The `limit` param is now set to `null` (no limit) by default, rather than 100.

#### Setting Params to Arrays

If you want to set a param value to an array, you now **must** type out the array brackets.

```twig
Old:
{% set query = craft.entries()
  .relatedTo('and', 1, 2, 3) %}

New:
{% set query = craft.entries()
  .relatedTo(['and', 1, 2, 3]) %}
```

#### Cloning Element Queries

In Craft 2, each time you call a parameter-setter method (e.g. `.type('article')`), the method would:

1. clone the `ElementCriteriaModel` object
2. set the parameter value on the cloned object
3. return the cloned object

That made it possible to execute variations of an element query, without affecting subsequent queries. For example:

```twig
{% set query = craft.entries.section('news') %}
{% set articleEntries = query.type('article').find() %}
{% set totalEntries = query.total() %}
```

Here `.type()` is applying the `type` parameter to a _clone_ of `query`, so it had no effect on `query.total()`, which will still return the total number of News entries, regardless of their entry types.

This behavior has changed in Craft 3, though. Now any time you call a parameter-setter method, the method will:

1. set the parameter value on the current element query
2. return the element query

Which means in the above code example, `totalEntries` will be set to the total _Article_ entries, as the `type` parameter will still be applied.

If you have any templates that count on the Craft 2 behavior, you can fix them using the [clone()](dev/functions.md#clone-object) function.

```twig
{% set query = craft.entries.section('news') %}
{% set articleEntries = clone(query).type('article').all() %}
{% set totalEntries = query.count() %}
```

### Query Methods

The `findElementAtOffset()` element query method has been removed in Craft 3. Use `nth()` instead.

Some element query methods have been renamed in Craft 3. The old methods have been deprecated, but will continue to work until Craft 4.

| Old Method      | New Method
| --------------- | --------------------------------------------------------
| `ids(criteria)` | `ids()` (setting criteria params here is now deprecated)
| `find()`        | `all()`
| `first()`       | `one()`
| `last()`        | `inReverse().one()` _(see [last()](#last))_
| `total()`       | `count()`

### Treating Queries as Arrays

Support for treating element queries as if they’re arrays has been deprecated in Craft 3, and will be completely removed in Craft 4.

When you need to loop over an element query, you should start explicitly calling `.all()`, which will execute the database query and return the array of results:

```twig
Old:
{% for entry in craft.entries.section('news') %}...{% endfor %}
{% for asset in entry.myAssetsField %}...{% endfor %}

New:
{% for entry in craft.entries.section('news').all() %}...{% endfor %}
{% for asset in entry.myAssetsField.all() %}...{% endfor %}
```

When you need to get the total number of results from an element query, you should call the `.count()` method:

```twig
Old:
{% set total = craft.entries.section('news')|length %}

New:
{% set total = craft.entries.section('news').count() %}
```

Alternatively, if you already needed to fetch the actual query results, and you didn’t set the `offset` or `limit` params, you can use the [length](https://twig.symfony.com/doc/2.x/filters/length.html) filter to find the total size of the results array without the need for an extra database query.

```twig
{% set entries = craft.entries()
  .section('news')
  .all() %}
{% set total = entries|length %}
```

### `last()`

`last()` was deprecated in Craft 3 because it isn’t clear that it needs to run two database queries behind the scenes (the equivalent of `query.nth(query.count() - 1)`).

In most cases you can replace calls to `.last()` with `.inReverse().one()` and get the same result, without the extra database query. (`inReverse()` will reverse the sort direction of all of the `ORDER BY` columns in the generated SQL.)

```twig
{# Channel entries are ordered by `postDate DESC` by default, so this will swap
   it to `postDate ASC`, returning the oldest News entry: #}

{% set oldest = craft.entries()
  .section('news')
  .inReverse()
  .one() %}
```

There are two cases where `inReverse()` won’t work as expected, though:

- when there is no `ORDER BY` clause in the SQL, therefore nothing to reverse
- when the `orderBy` param contains a <yii2:yii\db\Expression> object

In those cases, you can just replace the `.last()` call with what it’s already doing internally:

```twig
{% set query = craft.entries()
  .section('news') %}
{% set total = query.count() %}
{% set last = query.nth(total - 1) %}
```

## Elements

Tag elements no longer have a `name` property. Use `title` instead.

All elements’ `locale` properties have been deprecated, and will be completely removed in Craft 4. Use `siteId` if you need to know an element’s site ID, `site.handle` if you need to know its handle, or `site.language` if you need to know its site’s language.

## Models

Models’ `getError('attribute')` methods have been deprecated, and will be completely removed in Craft 4. Use `getFirstError('attribute')` instead.

## Locales

Some locale methods have been deprecated in Craft 3, and will be completely removed in Craft 4:

| Old Method         | What to do instead
| ------------------ | ------------------------------------
| `getId()`          | `id`
| `getName()`        | `getDisplayName(craft.app.language)`
| `getNativeName()`  | `getDisplayName()`

## Request Params

Your front-end `<form>`s and JS scripts that submit to a controller action will need to be updated with the following changes.

### `action` Params

`action` params must be rewritten in `kebab-case` rather than `camelCase`.

```twig
Old:
<input type="hidden" name="action" value="entries/saveEntry">

New:
<input type="hidden" name="action" value="entries/save-entry">
```

Some controller actions have been renamed:

| Old Controller Action       | New Controller Action
| --------------------------- | --------------------------
| `categories/createCategory` | `categories/save-category`
| `users/validate`            | `users/verify-email`
| `users/saveProfile`         | `users/save-user`
| `users/forgotPassword`      | `users/send-password-reset-email`

### `redirect` Params

`redirect` params must be hashed now.

```twig
Old:
<input type="hidden" name="redirect" value="foo/bar">

New:
<input type="hidden" name="redirect" value="{{ 'foo/bar'|hash }}">
```

The `redirectInput()` function is provided as a shortcut.

```twig
{{ redirectInput('foo/bar') }}
```

Some `redirect` param tokens have been renamed:

| Controller Action               | Old Token     | New Token
| ------------------------------- | ------------- | ---------
| `entries/save-entry`            | `{entryId}`   | `{id}`
| `entry-revisions/save-draft`    | `{entryId}`   | `{id}`
| `entry-revisions/publish-draft` | `{entryId}`   | `{id}`
| `fields/save-field`             | `{fieldId}`   | `{id}`
| `globals/save-set`              | `{setId}`     | `{id}`
| `sections/save-section`         | `{sectionId}` | `{id}`
| `users/save-user`               | `{userId}`    | `{id}`

### CSRF Token Params

CSRF protection is enabled by default in Craft 3. If you didn’t already have it enabled (via the `enableCsrfProtection` config setting), each of your front-end `<form>`s and JS scripts that submit to a controller action will need to be updated with a new CSRF token param.

```twig
{% set csrfParam = craft.app.request.csrfParam %}
{% set csrfToken = craft.app.request.csrfToken %}
<input type="hidden" name="{{ csrfParam }}" value="{{ csrfToken }}">
```

The `csrfInput()` function is provided as a shortcut.

```twig
{{ csrfInput() }}
```

## Plugins

See [Updating Plugins for Craft 3](extend/updating-plugins.md).
