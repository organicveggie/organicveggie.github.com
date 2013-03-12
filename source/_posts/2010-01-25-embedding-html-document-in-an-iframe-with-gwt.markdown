---
layout: post
title: "Embedding HTML Document in an IFRAME with GWT"
date: 2010-01-25 16:10
comments: true
categories: 
 - GWT
 - Google Web Toolkit
 - Java
 - Javascript
 - Programming
 - Web Development
---
While working on a recent project in GWT, I needed to embed a full HTML document inside an IFRAME. And I didn't want to specify a remote URL for the IFRAME - I actually wanted to shove the HTML content directly into the IFRAME.
<!-- more -->
Initially I thought it was trivial: create an `IFrameElement` and call `setInnerHTML`.

{% codeblock lang:javascript %}
IFrameElement iframe = Document.get().createIFrameElement();
iframe.setInnerHTML(htmlContent);
{% endcodeblock %}

Unfortunately, that doesn't work. It doesn't cause any errors, but it doesn't actually fill in the iframe. Instead, you have to use native javascript to write into the document object for the iframe:

{% codeblock lang:javascript %}
private final native void fillIframe(IFrameElement iframe, String content) {
  var doc = iframe.document;
 
  if(iframe.contentDocument)
    doc = iframe.contentDocument; // For NS6
  else if(iframe.contentWindow)
    doc = iframe.contentWindow.document; // For IE5.5 and IE6
 
   // Put the content in the iframe
  doc.open();
  doc.writeln(content);
  doc.close();
};
{% endcodeblock %}

Voila! However, one of the side effects is that the iframe doesn't include any CSS unless your embedded HTML references a stylesheet. If you want to manually add a reference to a specific stylesheet, you can do that through native javascript as well:

{% codeblock lang:javascript %}
private final native void addHeadElement(IFrameElement iframe, String cssUrl) {
  setTimeout(function() {
 
    var body;
    if ( iframe.contentDocument ) { 
      // FF
      iframe.contentDocument.designMode= "On";
      iframe.contentDocument.execCommand('styleWithCSS',false,'false');
      body= iframe.contentDocument.body;
    }
    else if ( iframe.contentWindow ) {
      // IE
      body = iframe.contentWindow.document.body;  
    }
 
    if (body == null) {
      return;
    }
 
    body.className = "custom-body-classname";
    var head = body.previousSibling;
    if(head == null) {
      head = iframe.contentWindow.document.createElement("head");
 
      iframe.contentWindow.document.childNodes[0].insertBefore(head, body);
    }
    var fileref = iframe.contentWindow.document.createElement("link");
    fileref.setAttribute("rel", "stylesheet");
    fileref.setAttribute("type", "text/css");
    fileref.setAttribute("href", cssUrl); 
    head.appendChild(fileref);
  }, 50);
};}
{% endcodeblock %}

There's still one more problem. You can't use either of native methods until the IFRAME element has been attached to the DOM. The easiest way around this is to add the IFRAME element to a panel and over the onLoad() method for the panel:

{% codeblock lang:javascript %}
final IFrameElement iframe = Document.get().createIFrameElement();
FlowPanel innerBox = new FlowPanel() {
    @Override
    protected void onLoad() {
        super.onLoad();
 
        // Fill the IFrame with the content html
        fillIframe(iframe, contentHtml);
 
        // Add a HEAD element to the IFrame with the appropriate CSS
        addHeadElement(iframe, cssUrl);
    }
};
innerBox.getElement().appendChild(iframe);
{% endcodeblock %}

I got the idea and the important code from an article titled [Inject HTML into an IFrame](http://softwareas.com/injecting-html-into-an-iframe) from [Software As She's Developed](http://softwareas.com/).