---
layout: post
title: "Converting diacritics to ASCII in Shibboleth IdP attributes"
description: ""
category: Devel
tags: [Shibboleth]
---

Some languages use [diacritical marks](https://en.wikipedia.org/wiki/Diacritic) to change the sound-value of the letter to which they are added. For example, this is a special "lorem ipsum"-like sentence in Czech language, where all available diacritical marks are used:

> Příliš žluťoučký kůň úpěl ďábelské ódy.

Sometimes there may be a requirement, that the value of a user's attribute released by a Shibboleth IdP has to be in ASCII. If the source attribute (as it is stored in the user database) is in UTF-8 and contains diacritics, one possible solution is to convert it using a [script attribute definition](https://wiki.shibboleth.net/confluence/display/SHIB2/ResolverScriptAttributeDefinition) in the Shibboleth IdP configuration.

For example, if we need the user's common name attribute in ASCII, we can add the following attribute definition to the `attribute-resolver.xml` configuration file:

```xml
<resolver:AttributeDefinition xsi:type="ad:Script" id="commonNameASCII">
    <resolver:Dependency ref="commonName" />  

    <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="http://example.org/attributes/commonName#ASCII" friendlyName="commonNameASCII" />

    <ad:ScriptFile>/opt/idp/script/commonNameASCII.js</ad:ScriptFile>

</resolver:AttributeDefinition>
```

The file `/opt/idp/script/commonNameASCII.js` contains the script, that actually does the conversion:

```javascript
importPackage(Packages.edu.internet2.middleware.shibboleth.common.attribute.provider);
importPackage(Packages.java.lang);
importPackage(Packages.java.text);

commonNameASCII = new BasicAttribute("commonNameASCII");

if (!commonName.getValues().isEmpty()) {
    originalValue = commonName.getValues().get(0);
    asciiValue = Normalizer.normalize(originalValue, Normalizer.Form.NFD).replaceAll("\\p{InCombiningDiacriticalMarks}+", "");

    commonNameASCII.getValues().add(asciiValue);
}
```
