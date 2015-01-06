---
layout: post
title: Mocking Log4j Logger
summary: ... is that even possible?
categories: java mock log4j mockito
tags: java mock log4j mockito
---

Generally I would argue that it is not the best idea to mock logger but in some special cases it could be necessary.

Recently I encountered one of those in a Java project. I had to mock a [log4j][1] logger with [mockito][2] and came up with the following code.

{% highlight java %}
import org.apache.log4j.LogManager;
import org.apache.log4j.Logger;
import org.apache.log4j.spi.LoggerRepository;
import org.apache.log4j.spi.RepositorySelector;
import org.mockito.invocation.InvocationOnMock;
import org.mockito.stubbing.Answer;

import static org.mockito.Matchers.anyString;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

public class LoggerTestUtils {

    public static Logger mockLogger(Object obj) {
        Logger logger = mock(Logger.class);
        mockLoggerForClass(logger, obj.getClass());
        return logger;
    }

    public static void mockLoggerForClass(final Logger logger, final Class clazz) {
        final LoggerRepository actualLoggerRepository = LogManager.getLoggerRepository();
        final String classNameToMock = clazz.getName();

        LoggerRepository loggerRepository = mock(LoggerRepository.class);
        RepositorySelector repositorySelector = mock(RepositorySelector.class);

        when(repositorySelector.getLoggerRepository()).thenReturn(loggerRepository);
        when(loggerRepository.getLogger(anyString())).then(new Answer<Logger>() {
            public Logger answer(InvocationOnMock invocation) throws Throwable {
                String className = (String) invocation.getArguments()[0];
                return className.equals(classNameToMock) ? logger : actualLoggerRepository.getLogger(className);
            }
        });

        LogManager.setRepositorySelector(repositorySelector, null);
    }

}
{% endhighlight %}

Please note that the original `LoggerRepository` has to be cached so that requests for other logger than the one to mock can still be fulfilled.
Also the logger will stay mocked for the whole run of the JVM but that should not be a problem in the tests runs.

And here is how one would use the mock.

{% highlight java %}
import org.apache.log4j.Logger;
import org.junit.Before;
import org.junit.Test;

import static LoggerTestUtils.mockLogger;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.verify;

public class SomeServiceTest {

    private Logger logger;
    private SomeService service;

    @Before
    public void setUp() throws Exception {
        someService = new SomeService();
        logger = mockLogger(scomeService);
    }

    @Test
    public void foo_onError_should
        someService.foo();
        verify(logger).error(anyString(), any(RuntimeException.class));
    }

{% endhighlight %}

In conclusion this shows that a look into the source code of the used libraries and frameworks is always a good thing.
You should know the tools you use and reading the source is a good and educational way to do so.

[1]: https://logging.apache.org/log4j/2.x/manual/plugins.html
[2]: https://code.google.com/p/mockito/

