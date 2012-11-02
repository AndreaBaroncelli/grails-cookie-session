# Cookie Session V2 Grails Plugin

The Cookie Session V2 plugin allows grails applications to store session data in cookies. By storing session data
in cookies, a complete record of a client's state is transmitted with each request from client to server.
This allows the application to be completely stateless and thus support drastically simplified scaling architectures and
robust fault tolerance.

## Simplified Scaling
After installing the cookie session v2 plugin, application scaled by starting instances 
of the application and routing requests requests to them. No clustering configuration, session replication or sticky sessions needed.

## Fault Tolerance
In the event an application instance or its server becomes unavailable, other application instances
are able to service the requests. If only a single instance of an application is deployed, it can be brought
down and up again without loosing clients' sessions because sessions are restored on each request. A side affect of this
architecture is that applications can be upgraded without dropping clients sessions. 

## Relationship to grails-cookie-session plugin
This project started as a fix to the grails-cookie-session plugin. However after
sorting through architectural issues and attempting to fix the code so that it supported my use-cases, the
project became a complete rewrite. With that said, this project would not have been possible (or at least would have
taken much longer!) had it not been for the original work. Many thanks to Masatoshi Hayashi
for giving me a place to start.

## Supported Use-cases
the driving motivation for this plugin, was to use cookie-based session with full support for the following
  <ul>
    <li>flash scope</li>
    <li>webflow</li>
    <li>secure session</li>
    <li>sessions larger than 8kb</li>
  </ul>

## Secure Sessions
this plugin can be configured to encrypt the serialized session stored in the cookie. This feature prevents
clients from accessing or tampering with potentially sensitive data being stored in the session.

# Installation

grails install-plugin cookie-session-v2

# Issues

# Configuration

<table>
  <thead>
    <tr>
      <th>name</th>
      <th width="25%">default</th>
      <th>description</th>
  </thead>
  <tbody>
    <tr>
      <td>grails.plugin.cookiesession.enabled</td>
      <td>true</td>
      <td>enables or disables the cookie session.</td>
    </tr>
    <tr>
      <td>grails.plugin.cookiesession.encryptcookie</td>
      <td>true</td>
      <td>enable or disable encrypting session data stored in cookies.</td>
    </tr>
    <tr>
      <td>grails.plugin.cookiesession.cryptoalgorithm</td>
      <td>Blowfish</td>
      <td>The cryptographic algorithm used to encrypt session data (i.e. Blowfish, DES, DESEde, AES). NOTE: your secret must be compatible with the crypto algorithm.</td>
    </tr>
    <tr>
      <td>grails.plugin.cookiesession.secret</td>
      <td>randomly generated at runtime</td>
      <td>The secret key used to encrypt session data. If not set, a random key will be created at runtime. NOTE: if multiple instances of the application are seriving requests, this parameter must be set manually so that all instances can decrypt sessions produced by any instance of the application.</td>
    </tr>
    <tr>
      <td>grails.plugin.cookiesession.cookiecount</td>
      <td>5</td>
      <td>The maximum number of cookies that are created to store the session in</td>
    </tr>
    <tr>
      <td>grails.plugin.cookiesession.maxcookiesize</td>
      <td>2048</td>
      <td>The max size for each cookie expressed in bytes.</td>
    </tr>
    <tr>
      <td>grails.plugin.cookiesession.sessiontimeout</td>
      <td>0</td>
      <td>The length of time a session can be inactive for expressed in seconds. Zero indicates that a session will be active for as long as the browser is open.</td>
    </tr>
    <tr>
      <td>grails.plugin.cookiesession.cookiename</td>
      <td>gsession-X</td>
      <td>X number of cookies will be written per the cookiecount parameter. Each cookie is suffixed with the integer index of the cookie.</td>
    </tr>
  </tbody>
</table>

### Enabling large session
To enable large sessions, you'll need to increase the max http header size of the connector. In tomcat, this can be configured in the server.xml with the maxHttpHeaderSize. Set this value to something large such as 262144 (i.e. 256kb). 

When developing in grails, you can configure the embedded tomcat instance with the tomcat startup event.
<ol>
  <li>create the file scripts/_Events.groovy in your project directory</li>
  <li>Add the following code:
    <code>
      eventConfigureTomcat = {tomcat ->
        tomcat.connector.setAttribute("maxHttpHeaderSize",262144)
      }
    </code>
  </li>
</ol>

### Enabling webflow hibernate session
To enable webflows to correctly serialize and deserialize hibernate sessions, some additional configuration is needed. The following instructions show
how to explicitly name the hibernate session factory.  
<ol>
  <li>create the hibernate.cfg.xml file: grails create-hibernate-cfg-xml</li>
  <li>
    edit the grails-app/conf/hibernate/hibernate.cfg.xml file and add the following line under the &lt;session-factory&gt; element
    <code>
      &lt;property name="hibernate.session_factory_name"&gt;session_factory&lt;/property&gt;
    </code>
  </li>
</ol>

## Logging

## Example
<code>
grails.plugin.cookiesession.enabled = true <br>
grails.plugin.cookiesession.encryptcookie = true <br>
grails.plugin.cookiesession.cryptoalgorithm = "Blowfish" <br>
grails.plugin.cookiesession.secret = "This is my secret. There are many like it, but this one is mine.".bytes <br>
grails.plugin.cookiesession.cookiecount = 10 <br>
grails.plugin.cookiesession.maxcookiesize = 2048  // 2kb <br>
grails.plugin.cookiesession.sessiontimeout = 3600 // one hour<br>
grails.plugin.cookiesession.cookiename = 'gsession' <br>
</code>
