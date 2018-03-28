---
layout: post
title: "Custom Quill.js To Add More Attributes To Image"
date: 2018-03-28 15:02
comments: true
categories: ["javascript"]
---

Quill.js image default only allow 'src', 'alt' 'height' and 'width' attribute.  
Here are the solutions to add more attributes to image when initializing from fulltext html.  

Solution1:
{% codeblock lang:javascript %}
class ImageBlot extends Image {
  static create(value) {
    if (typeof value == 'string') {
      return super.create(value);
    } else {
      return value;
    }
  }

  static value(domNode) {
    return domNode;
  }
}
Quill.register(ImageBlot);
{% endcodeblock %}

Solution2:
{% codeblock lang:javascript %}
class ImageBlot extends Image {
  static get ATTRIBUTES() {
    return [ 'alt', 'height', 'width', 'class', 'data-original', 'data-width', 'data-height', 'style-data' ]
  }

  static formats(domNode) {
    return this.ATTRIBUTES.reduce(function(formats, attribute) {
      if (domNode.hasAttribute(attribute)) {
        formats[attribute] = domNode.getAttribute(attribute);
      }
      return formats;
    }, {});
  }

  format(name, value) {
    if (this.constructor.ATTRIBUTES.indexOf(name) > -1) {
      if (value) {
        this.domNode.setAttribute(name, value);
      } else {
        this.domNode.removeAttribute(name);
      }
    } else {
      super.format(name, value);
    }
  }
}
Quill.register(ImageBlot);
{% endcodeblock %}

You can specify the whitelist for attributes with solution2.  

The quill.js depends on parchment and quill-delta.  
Parchment is Quill's document model. You can find Parchment.Embed here.  
Deltas are a simple, yet expressive format that can be used to describe contents and changes.  

I find the steps of initializing after diving into the source code and some debug:  
constructor in core/quill.js -> clipboard.convert -> clipboard.matchBlot -> setContents -> applyDelta -> insertAt and formatAt  
The matchBlot is key step for my solutions. It explains how quill compose the delta instance.  

    let value = match.value(node);
    embed[match.blotName] = value;
    delta = new Delta().insert(embed, match.formats(node));

https://github.com/quilljs/quill/

