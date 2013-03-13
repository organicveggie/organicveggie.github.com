---
layout: post
title: "Struts Redirect w/Parameters"
date: 2006-01-23 23:00
comments: true
categories: 
 - Java
 - Programming
 - Struts
 - Web Development
---
If you have a Struts Action servlet and you want to redirect to another page, the standard Struts technique is to return an ActionForward and setup an appropriate forward entry in `struts-config.xml`:

{% codeblock lang:java %}
return mapping.findForward("success");
{% endcodeblock %}

Unfortunately, this doesn’t provide a mechanism for passing request parameters to the target. So what can you do if you want to redirect to another page and pass some parameters along? You use an additional class:

{% codeblock ForwardParameters.java %}
public class ForwardParameters
{
  private Map params = null;
 
  public ForwardParameters()
  {
    params = new HashMap();
  }
 
  /**
   * Add a single parameter and value.
   *
   * @param paramName     Parameter name
   * @param paramValue    Parameter value
   *
   * @return A reference to this object.
   */
  public ForwardParameters add(String paramName, String paramValue)
  {
    params.put(paramName,paramValue);
    return this;
  }
 
  /**
   * Add a set of parameters and values.
   *
   * @param paramMap  Map containing parameter names and values.
   *
   * @return A reference to this object.
   */
  public ForwardParameters add(Map paramMap)
  {
    Iterator itr = paramMap.keySet().iterator();
    while (itr.hasNext())
    {
      String paramName = (String) itr.next();
      params.put(paramName, paramMap.get(paramName));
    }
 
    return this;
  }
 
  /**
   * Add parameters to a provided ActionForward.
   *
   * @param forward    The ActionForward object to add parameters to.
   *
   * @return ActionForward with parameters added to the URL.
   */
  public ActionForward forward(ActionForward forward)
  {
    StringBuffer path = new StringBuffer(forward.getPath());
 
    Iterator itr = params.entrySet().iterator();
    if (itr.hasNext())
    {
      //add first parameter, if avaliable
      Map.Entry entry = (Map.Entry) itr.next();
      path.append("?" + entry.getKey() + "=" + entry.getValue());
 
      //add other parameters
      while (itr.hasNext())
      {
        entry = (Map.Entry) itr.next();
        path.append("&amp;" + entry.getKey() + "=" + entry.getValue());
      }
    }
 
    return new ActionForward(path.toString());
  }
}
{% endcodeblock %}

Here are some examples of using ForwardParameters:

Long form:

{% codeblock lang:java %}
ActionForward forward = mapping.findForward("success");
ForwardParameters fwdParams = new ForwardParameters();
fwdParams.add("myparam", "myvalue");
fwdParams.add("something", "fornothing");
return fwdParams.forward(forward);
{% endcodeblock %}

Concise form:

{% codeblock lang:java %}
ActionForward forward = mapping.findForward("success");
return new ForwardParameters().add("myparam", "myvalue").add("something", "fornothing").forward(forward);
{% endcodeblock %}

Ultra concise form:

{% codeblock lang:java %}
return new ForwardParameters().add("myparam", "myvalue").add("something", "fornothing").forward(mapping.findForward("success"));
{% endcodeblock %}

This is based on a suggestion I found on [Experts-Exchange](http://www.experts-exchange.com/) by [dualsoul](http://www.experts-exchange.com/Programming/Programming_Languages/Java/Q_20830927.html).

---

Edit June 5th, 2006:

As someone just pointed out to me this morning, you can use the `ActionRedirect` class instead of my `ForwardParameters` class. According to the JavaDocs, `ActionRedirect` was added in Struts 1.2.7 and is:

{% blockquote JavaDocs %}
... a subclass of ActionForward which is designed for use in redirecting requests, with support for adding parameters at runtime.
{% endblockquote %}

