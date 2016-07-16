---
layout: post
title: "WebElement Waits"
intro: "In this short tutorial we will extend Selenium WebDriver API to apply wait conditions on WebElement objects. "
---

In this short tutorial we will extend Selenium WebDriver API to apply wait conditions on [WebElement](http://google.com) objects. 

Selenium Webdriver 2.0 library comes with a set of utilities for applying different wait conditions when locating DOM elements or when waiting for an event to happen in DOM.

We generally use [WebDriverWait](http://google.com) and [ExpectedCondition](http://google.com) objects for applying busy-wait-polling machnism when locating elements via WebDriver object. The polling behavior can be customized via different parameters such as wait timeout, interval between polls and which exceptions to ignore during polling. It also provides [ExpectedConditions](http://google.com) - a helper static class which includes many ready-made ExpectedCondition implementations commonly used in WebDriver tests. 

However, I had several scenarios where I wanted to apply an ExpectedCondition on a WebElement object, not on a WebDriver object. For example, finding nested elements of a WebElement object such as table rows. Such elements might be loaded dynamicaly, thus using table.findElements(rowsXpath) won't be reliable enough.

WebElement interface extends [SearchContext](http://google.com) interface, so it seemed abvious that library will support WebElement waits. But current implementation doesn't provide such support.

Since WebDriverWait is based on a generic type object [FluentWait](http://google.com), implementing our WebElementWait should be straight forward.

WebElementWait will subclass FluentWait<T> and will define WebElement class as its input type:

{% highlight java %}
public class WebElementWait extends FluentWait<WebElement> {

    public WebElementWait(WebElement element, long timeOutInSeconds) {
        super(element);
        withTimeout(timeOutInSeconds, TimeUnit.SECONDS);
    }

}
{% endhighlight %}

Once WebElementWait is ready, we can implement WebElement-specific ExpectedConditions.

First, we define a new interface for ExpectedCondition of WebElement type

{% highlight java %}
public interface WebElementExpectedCondition<T> extends Function<WebElement, T> {}
{% endhighlight %}

Finally, we can implement our desired ExpectedCondition-s based on new interface. For example:

{% highlight java %}
/**
 * An expectation for checking that an element is present on the DOM of a page
 * and visible. Visibility means that the element is not only displayed but
 * also has a height and width that is greater than 0.
 *
 * @param locator used to find the element
 * @return the WebElement once it is located and visible
 */

public static WebElementExpectedCondition<WebElement> visibilityOfElementLocated(
        final By locator) {

    return new WebElementExpectedCondition<WebElement>() {
        @Override
        public WebElement apply(WebElement element) {
            try {
                WebElement innerElement = element.findElement(locator);
                return innerElement.isDisplayed() ? innerElement : null;
            } catch (StaleElementReferenceException e) {
                return null;
            }
        }

        @Override
        public String toString() {
            return "visibility of element located by " + locator;
        }
    };
}
{% endhighlight %}

Combining all pieces together, we can apply the wait condition on a WebElement object:

{% highlight java %}

public List<WebElement> getTableRows(WebElement table) {

    WebElementWait wait = new WebElementWait(table, 30);
    wait.ignoring(NoSuchElementException.class);
    element = wait.until(WebElementExpectedConditions.visibilityOfElementLocated(rowsXpath));
    
}
{% endhighlight %}

For full source code and othere examples, you can refer to selenium-toolbox repository on Github.
