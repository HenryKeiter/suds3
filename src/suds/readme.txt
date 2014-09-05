This package is a very rough port of suds 0.4 to Python 3.

The procedure for recreating the basic port is described below (adapted from https://fedorahosted.org/suds/ticket/393). However, other changes have been applied to this module, which are not recorded in this file. See the git history.

1. Manually apply the following changes:
    Find all "__unicode__" methods
        - eliminate them all and adapt __str__  to compensate (a few places require slight modification, but mostly the body of __unicode__ just replaces the body of __str__).

    ./transport/__init__.py
        Change (2x) "s.append(self.message)" to s.append(str(self.message))

    ./transport/http.py
        remove    "from urlparse import urlparse"
        remove    "urllib2"
        add       "import urllib.request"
        add       "import urllib.error"
        change    "u2.HTTPError" to "urllib.error.HTTPError"
        change    "u2.Request" to "urllib.request.Request"
        change    "u2.build_opener" to "urllib.request.build_opener"
        change    "u2.ProxyHandler" to "urllib.request.ProxyHandler"
        change    "u2.__version__" to "urllib.request.__version__"
        u2opener method ->
            change "u2.Request.build_opener(*self.u2handlers())" to "urllib.request.build_opener()"
        send method ->
            change "url = request.url" to "url = request.url.decode()"
            change "result = Reply(200, fp.headers.dict, fp.read())" to "result = Reply(200, fp.headers, fp.read())"

    ./transport/https.py
        change "import urllib2 as u2" to "import urllib2"
        change "u2.HTTPBasicAuthHandler" to "urllib2.HTTPBasicAuthHandler"

    ./sudsobject.py
        remove "from new import classobj"
        change "subclass = classobj(name, bases, dict)" to "subclass = type(name, bases, dict)"
        subclass method ->
            remove "name = name.encode('utf-8')"

    ./servicedefinition.py
        remove "self.types.sort(cmp=tc)"

    ./reader
        change "content = ctx.document" to "content = ctx.document.decode()"

    ./wsdl.py
        remove "import re, soaparray"
        add "import re"
        add "import soaparray"

    ./bindings/binding.py
        change (2x) "reply = self.replyfilter(reply)" to "reply = self.replyfilter(reply).decode()"

    ./xsd/sxbase.py
        __repr__ method ->
            Remove ".encode('utf-8')" from return statement

    ./sax/date.py
        change "return '{0}{1:+02d}:00'.format(time, self.tz.local)" to "return '{0}{1:+03d}:00'.format(time, self.tz.local)"
        change "LOCAL = ( 0-time.timezone/60/60 )" to "LOCAL = ( 0-time.timezone//60//60 )"

2. Run the 2to3 tool.
    python C:\Python33\Tools\Scripts\2to3.py -n -w C:\Python33\Lib\site-packages\suds
