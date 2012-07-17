django-unhosted: [Unhosted](http://unhosted.org/) [remoteStorage](http://www.w3.org/community/unhosted/wiki/RemoteStorage) server implementation
--------------------

Under development, not yet ready for general usage.


Deploy
--------------------

(for testing/development only at the moment, hence that compicated)

	git clone https://github.com/mk-fg/django-unhosted
	git clone https://github.com/unhosted/remoteStorage.js
	wget http://twitter.github.com/bootstrap/assets/bootstrap.zip
	wget http://requirejs.org/docs/release/jquery-require/jquery1.7.2-requirejs2.0.4/jquery-require-sample.zip
	unzip jquery-require-sample.zip

	basepath=$(pwd)
	pushd django_unhosted/django_unhosted/static
	unzip $basepath/bootstrap.zip
	cd demo
	ln -s $basepath/remoteStorage.js/src remoteStorage
	ln -s $basepath/jquery-require-sample/webapp/scripts/require-jquery.js .
	popd

settings.py:

	STATIC_ROOT = ...
	STATIC_URL = ...

	TEMPLATE_LOADERS = (
		...
		'django_unhosted.apps.webfinger.xrd_gen.Loader',
	)

	TEMPLATE_CONTEXT_PROCESSORS = (
		...
		'django.core.context_processors.request',
	)

	INSTALLED_APPS = (
		...
		'django_unhosted',
		'oauth2app',
		'crispy_forms',
		'south',
	)

	CRISPY_TEMPLATE_PACK = 'bootstrap'
	CRISPY_FAIL_SILENTLY = not DEBUG


Customization
--------------------

##### Webfinger

If [webfinger](https://tools.ietf.org/html/draft-jones-appsawg-webfinger-01) and
[host-meta](https://tools.ietf.org/html/draft-hammer-hostmeta-05) requests for
the domain should carry more data than just for remoteStorage, they can be
extended either by replacing webfinger app entirely or adding custom templates
for it.

Webfinger app is using "webfinger/host_meta.{xml,json}" and
"webfinger/webfinger.{xml,json}" templates, provided by
django_unhosted.apps.webfinger.xrd_gen.Loader or generated dynamically (in case
of json, if template provide can't be found).

See example xml templates in
[django_unhosted/templates/webfinger/{host_meta,webfinger}.xml.example]
(https://github.com/mk-fg/django-unhosted/blob/master/django_unhosted/templates/webfinger/).


Known issues
--------------------

These are implementation-related issues, not the issues with the protocols
themselves (which doesn't imply there's none of the latter, just that it's not a
place for them).

##### Webfinger

* No easy support for [signed
	XRD](http://docs.oasis-open.org/xri/xrd/v1.0/xrd-1.0.html#signature), due to
	lack of support in [python-xrd](https://github.com/jcarbaugh/python-xrd/),
	signed *static* xml "templates" (or just files, served from httpd) can be used
	as a workaround if TLS is not an option.

* [Original (PyPI) version](https://github.com/jcarbaugh/python-xrd/) of
	python-xrd can't be used, due to lack of support for custom attributes for
	links (like "auth" and "api"), which are used in remoteStorage protocol atm,
	only [my fork](https://github.com/mk-fg/python-xrd/) with experimental support
	for these.
	**TODO:** work with python-xrd upstream on this.

##### OAuth2

* Stored object path (think "public/myphoto.jpg") is used as OAuth2 "scope" by
	remoteStorage.
	oauth2app basically keeps a single table of these (treating them as a finite
	up-front set of capabilities).

	Two problems here:

	* It stores "scope" as a 255-char key, while paths can potentially be longer
		(thus I hash them here, so they reliably fit into the field) and more
		numerous, resulting in possible hash collisions.
		I work arround this by abusing "description" field to store (and check) path
		there, but it's a hack.

	* There's some extra overhead involved for maintaining the - pointless in this
		case - table, involving counting references to the entries there and
		dropping them when it reaches zero (e.g. no one has "public/myphoto.jpg"
		path anymore).

	**TODO:** work with oauth2app upstream on this.
