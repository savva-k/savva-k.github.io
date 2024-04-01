---
title: "Generating PDF with Apache FOP and Maven. An example and some tricks."
date: "2019-02-13"
categories:
  - "Coding"
tags:
  - "Apache Fop"
  - "Java"
  - "PDF"
image: images/main.png
---

This January (and, actually, February) I worked on generating PDF files. During the investigation process, we've chosen [Apache FOP](https://xmlgraphics.apache.org/fop/) as a free powerful tool for this task.

Firstly I wanted to write a step by step guide on how to generate a PDF file from a Java object. But eventually, I decided to keep it short, as [the original documentation](https://xmlgraphics.apache.org/fop/quickstartguide.html) contains almost all you need.

So in this blog I'll describe some interesting moments that I met during the development phase.

Also, I've prepared an example project, feel free to clone and play around:  
[https://github.com/savva-k/apache-fop-example](https://github.com/savva-k/apache-fop-example)

#### Adding custom fonts to FOP

Adding fonts is pretty simple. I used an XML config, so I will show how to do that in XML:

```xml
<?xml version="1.0"?>
<fop version="1.0">
    <!-- Simple FOP XML config -->

    <base>.</base>
    <strict-validation>false</strict-validation>
    <strict-configuration>false</strict-configuration>
    <source-resolution>72</source-resolution>
    <target-resolution>72</target-resolution>
    <default-page-settings height="11.00in" width="8.50in"/>
    <renderers>
        <renderer mime="application/pdf">
            <fonts>
                <font kerning="yes" embed-url="path/to/Roboto-Regular.ttf" embedding-mode="subset">
                    <font-triplet name="Roboto" style="normal" weight="normal"/>
                </font>
                <font kerning="yes" embed-url="path/to/Roboto-Bold.ttf" embedding-mode="subset">
                    <font-triplet name="Roboto" style="normal" weight="Bold"/>
                </font>
            </fonts>
        </renderer>
    </renderers>
</fop>
```

Then we can use the font by setting the font-family attribute:

```xml
<fo:root font-family="Roboto">
    <someContent>
        This font is regular.
        <fo:inline font-weight="bold">This font is bold.</fo:inline>
    </someContent>
</fo:root>
```

#### How to use WOFF fonts with Apache FOP?

I didn't find the answer to this question. But I've found a good WOFF to OTF conversion tool! Here it is: [https://github.com/hanikesn/woff2otf](https://github.com/hanikesn/woff2otf)  
OTF fonts work fine with FOP.

#### How to access fonts from resources in a JAR?

At some time I realized that all the fonts are placed in a JAR file and other resources like images from external servers are not. To solve this problem I have created a custom ResourceResolver implementation:

```java
public final class CustomPathResolver implements ResourceResolver {
    private static final String FONTS_FOLDER = "/fonts/";

    private ResourceResolver defaultResourceResolver = ResourceResolverFactory.createDefaultResourceResolver();

    @Override
    public Resource getResource(URI uri) throws IOException {
        if (uri.toString().contains(FONTS_FOLDER)) {
            return new Resource(PdfGenerator.class.getResourceAsStream(FONTS_FOLDER + FilenameUtils.getName(uri.toString())));
        } else {
            return new Resource(uri.toURL().openStream());
        }
    }

    @Override
    public OutputStream getOutputStream(URI uri) throws IOException {
        return defaultResourceResolver.getOutputStream(uri);
    }
}
```

This ResourceResolver checks whether the requested resource is in the "fonts" folder and if so, we are looking for a resource in the classpath.  
To use this ResourceResolver we should manually specify it during the FopFactoryBuilder creation.

```java
DefaultConfigurationBuilder cfgBuilder = new DefaultConfigurationBuilder();
try {
    Configuration config = cfgBuilder.build(PdfGenerator.class.getResourceAsStream(CONFIG_FOP_XML));
    FopFactoryBuilder factoryBuilder = new FopFactoryBuilder(new File(CURRENT_DIR).toURI(), new CustomPathResolver()).setConfiguration(config);
    fopFactory = factoryBuilder.build();
} catch (Exception e) {
    // handling the error
}
```

#### Apache FOP cannot load images via HTTPS

In my case, the root cause was the SSLHandshakeException. My server didn't trust the server that hosted the image. I wouldn't do that neither!

This problem has at least three solutions: the best, a good one and the one I had to use.

**The best solution:** use a certificate issued by a well known CA on the server that hosts your images.

A good solution: [add the server's certificate to your keystore](https://docs.oracle.com/javase/tutorial/security/toolsign/rstep2.html).

The last solution is to disable checking certificate validity at all. **This is not recommended, as you become vulnerable to man-in-the-middle attacks.**  
To do this, you should implement a new TrustManager and a HostnameVerifier, initialize an SSLContext and set the HostnameVerifier to the HttpsURLConnection (which is used internally by Apache FOP).

```java
static {
    TrustManager[] trustAllCerts = new TrustManager[] {new X509TrustManager() {
        public java.security.cert.X509Certificate[] getAcceptedIssuers() {
            return null;
        }
        public void checkClientTrusted(X509Certificate[] certs, String authType) {
        }
        public void checkServerTrusted(X509Certificate[] certs, String authType) {
        }
        }
    };

    SSLContext sc = SSLContext.getInstance("SSL");
    sc.init(null, trustAllCerts, new java.security.SecureRandom());
    HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());

    HostnameVerifier allHostsValid = new HostnameVerifier() {
        public boolean verify(String hostname, SSLSession session) {
            return true;
        }
    };
    HttpsURLConnection.setDefaultHostnameVerifier(allHostsValid);
}
```

I think that's it and I hope this helps. 🙂
