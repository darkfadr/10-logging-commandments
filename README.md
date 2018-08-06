# The 10 Commandments of Logging



points

- he log is the source of truth and done correctly, they become the ultimate source of truth







## 1) Thou shalt not write log by yourself

Never, ever use printf or write your log entries by yourself to files, or handle log rotation by yourself. Please do your ops guys a favor and use a standard library or system API call for this.

This way, you’re sure the running application will play nicely with the other system components, will log to the right place or network services without special system configuration.

So, if you just use the system API, then this means logging with `syslog(3)`. Learn how to use it.

If you instead prefer to use a logging library, there are plenty of those especially in the Java world, like [Log4j](http://logging.apache.org/log4j/2.x/), [JCL](https://commons.apache.org/logging/guide.html), [slf4j](http://www.slf4j.org/) and [logback](http://logback.qos.ch/). My favorite is the combination of slf4j and logback because it is very powerful and relatively easy to configure (and allows JMX configuration or reloading of the configuration file).

Dive into transport flexibility.

## 2) Thou shalt log at the proper level

If you followed the 1st commandment, then you can use a different log level per log statement in your application. One of the most difficult task is to find at what level this log entry should be logged.

Here I’ve given some advice:

- *TRACE level*: this is [a code smell](http://www.lenholgate.com/blog/2005/07/no-thats-not-the-point-and-yes-trace-logging-is-bad.html) if used in production. This should be used during development to track bugs, but never committed to your VCS.
- *DEBUG level*: log at this level about anything that happens in the program. This is mostly used during debugging, and I’d advocate trimming down the number of debug statement before entering the production stage, so that only the most meaningful entries are left, and can be activated during troubleshooting.
- *INFO level*: log at this level all actions that are user-driven, or system specific (ie regularly scheduled operations…)
- *NOTICE level*: this will certainly be the level at which the program will run when in production. Log at this level all the notable event that are not considered an error.
- *WARN level*: log at this level all event that could potentially become an error. For instance if one database call took more than a predefined time, or if a in memory cache is near capacity. This will allow proper automated alerting, and during troubleshooting will allow to better understand how the system was behaving before the failure.
- *ERROR level*: log every error conditions at this level. That can be API calls that return errors or internal error conditions.
- *FATAL level*: too bad it’s doomsday. Use this very scarcely, this shouldn’t happen a lot in a real program. Usually logging at this level signifies the end of the program. For instance, if a network daemon can’t bind a network socket, log at this level and exit is the only sensible thing to do.

Note that the default running level in your program or service might widely vary. For instance, I run my server code at level *INFO* usually, but my desktop programs run at level *DEBUG*. This falls in line with commandment #8

## 3) Honor thy log category

Most logging library I cited in the 1st commandment allow to specify a logging category. This category allows to classify the log message, and will ultimately, based on the logging framework configuration, be logged in a distinct way or not logged at all.

Most of the time java developers use the fully qualified class name where the log statement appears as the category. This is a scheme that works relatively fine if your program respects the [simple responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

Log categories in Java logging libraries are hierarchical, so for instance logging with category `com.daysofwonder.ranking.ELORankingComputation` would match the top level category `com.daysofwonder.ranking`. This would allow the ops engineer to setup a logging configuration that works for all the ranking subsystem by just specifying configuration for this category. But it could at the same time, produce logging configuration for child categories if needed.

We can extend the paradigm a little bit further to help to troubleshoot the specific situation. Imagine that you are dealing with a server software that responds to user based request (like a REST API for instance). If your server is logging with this category `my.service.api.<apitoken>` (where apitoken is specific to a given user), then you could either log all the API logs by allowing `my.service.api` or a single misbehaving API user by logging with a more detailed level and the category `my.service.api.<bad-user-api-token>`. Of course, this requires a system where you can change logging configuration on the fly.

Also dive into how the category can be used to quickly pinpoint when in the code the issue occurred.

## 4) Thou shalt write meaningful logs

This might probably be the most important commandment. There’s nothing worst than cryptic log entries assuming you have a deep understanding of the program internals.

When writing your log entries messages, always anticipate that there are emergency situations where the only thing you have is the log file, from which you have to understand what happened. Doing it right might be the subtle difference between getting fired and promoted :)

When a developer writes a log message, it is in the context of the code in which the log directive is to be inserted. Under these conditions, we tend to write messages the infer on the current context. Unfortunately, when reading the log itself this context is absent, and those messages might not be understandable.

One way to overcome this situation (and that’s particularly important when writing at the warn or error level), is to add remediation information to the log message, or if not possible, what was the purpose of the operation, and it’s outcome.

Also, do not log message that depends on previous messages content. The reason is that those previous messages might not appear if they are logged in a different category or level, or worst can appear in a different place (or way before) in a multi-threaded or asynchronous context.

## 5) Thy log shalt be written in English



## 6) Thy shalt log with context

## 7) Thou shalt log in machine parseable format

## 8) Thou shalt not log too much or too little

http://www.lenholgate.com/blog/2003/05/death-by-debug-trace.html

## 9) Thou shalt think to the reader

## 10) Thou shalt not log only for troubleshooting

As the log messages are for a different audience, log messages will be used for different reasons. Even though troubleshooting is certainly the most evident target of log messages, you can also use log messages very efficiently for:

- *Auditing*: this is sometimes a business requirement. The idea is to capture significant events that matter to the management or legal people. These are statements that describe usually what users of the system are doing (like who signed-in, who edited that, etc…).
- *Profiling*: as logs are timestamped (sometimes to the millisecond level), it can become a good tool to profile sections of a program, for instance by logging the start and end of an operation, you can either automatically (by parsing the log) or during troubleshooting infer some performance metrics without adding those metrics to the program itself.
- *Statistics*: if you log each time a certain event happens (like a certain kind of error or event) you can compute interesting statistics about the running program (or the user behaviors). It’s also possible to hook this to an alert system that can detect too many errors in a row.



















## Sources

[Scalyr](https://blog.scalyr.com/2018/05/the-10-commandments-of-logging/)