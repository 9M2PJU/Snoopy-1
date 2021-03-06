# Introduction


# Requirements
## Python Packages
* [Beaker](http://beaker.readthedocs.org/)
* [cryptacular](http://pypi.python.org/pypi/cryptacular/)
* [Flask](http://flask.pocoo.org/)
* [MySQLdb](http://mysql-python.sourceforge.net/MySQLdb.html)
* [SQLAlchemy](http://sqlalchemy.org/)


# Plug-ins
Plug-ins consist of two parts:
* Back-end (data providing) part, written in Python
* Front-end (displaying) part, written in JavaScript (optional)

## Back-end
A plug-in is a callable that is dynamically loaded by the Snoopy application.
Let's start with an example:

    from snoopy import db, pluginregistry

    @pluginregistry.add('client-data', 'ssidlist', 'SSID List', js='/static/js/ssidlist.js')
    def ssid_list(mac):
        with db.SessionCtx() as session:
            query = session.query(
                distinct(db.Probe.probe_ssid) # SELECT DISTINCT probe_ssid FROM probes
            ).filter_by(
                device_mac=mac                # WHERE device_mac=$mac
            ).order_by(
                db.Probe.probe_ssid           # ORDER BY probe_ssid
            )
            return query.all()

Now for the blow-by-blow...

    from snoopy import db, pluginregistry

We need `db` to access the Snoopy database and `pluginregistry` to register our
plug-in.

    @pluginregistry.add('client-data', 'ssidlist', 'SSID List', js='/static/js/ssidlist.js')

This is how we register our plug-in (which can be any callable). The first
argument specifies the name of the group that this plug-in falls into. We
specified `client-data`, since this plug-in accepts a MAC address and returns
data to be displayed in a client data window. Apart from `client-data`, there
is also the `session-data` group of plug-ins that operate on proximity
sessions. The second parameter is the plug-in name. This name is for internal
use only and should be JSON-friendly. The third parameter is the plug-in title.
Essentially it is the same as the name, but the title is displayed in the GUI.
More specifically, this is the title of the data section in the client data
window.

All keyword arguments after the title are saved as options. While the plug-in
system does not require any options, Snoopy supports the `js` option. This
option specifies the **URL** (from the browser side, not from Snoopy's/the
server's side) of the JavaScript component of this plug-in.

    def ssid_list(mac):

This plug-in is a simple function. It accepts a MAC address string, as
per `client-data` plug-ins' definition. The name of the callable is not used.

The body of this example plug-in simply performs a SQLAlchemy query that
basically does `SELECT DISTINCT probe_ssid FROM probes WHERE device_mac=$mac`
and returns the results as a list. The return values of plug-ins are usually
added as part of a JSON response. Therefore the results has the limitation that
it must be JSON-friendly (as supported by Flask's `jsonify()` function).

As simple as that!

## Front-end
Front-end plug-ins for client data are referred to as client data handlers.
They are registered via the `Snoopy.registerPlugin(sectionName, dataHandler)`
function. `sectionName` is the name of the data section that the data handler
wants to handle. This name is the same as the back-end plug-in name that
generated the data. The `dataHandler` must be a function with the following
signature: `function($section, clientData) {...}`. `$section` is a jQuery
object of the div element where the data handler may display data or affect
changes. The client data (generated by the back-end plug-in) is given in the
`clientData` parameter.

Since the data handler's only allowed output is changes to the given div
(`$section`), no return value is necessary.
