---
title: Organizing Your Clojure Environment and Logs with Leiningen
---

Coming from a Rails environment, one of the things I took for granted was a nice logging setup in which
all logs were written to a log file for each environment name. Another was easily setting environment variables
through lines like:

```ruby
ENV["RAILS_ENV"] ||= "test"
```

While working on a Clojure project recently, I began to feel
the pain of not knowing where exactly my logs were being written, and not having my test environment automatically loaded
when running tests. But, by using just a few libraries we can create a nice environment for our
Clojure app that allows us to set logging, database, or any other configurations while simply running our app with Leiningen.

### No Logging

Let's start with a basic app with no logging or environment-specific settings:

```clj
(defn -main [& args]
  (println "Hello, World!"))
```

```bash
$ lein run
Hello World!
```

### Add Some Logging

We can add some basic logging with [clojure/tools.logging](https://github.com/clojure/tools.logging). We just add the
dependency to our ```project.clj```, require it in our app, and log some info:

```clj
:dependencies [[org.clojure/clojure "1.5.1"]
               [org.clojure/tools.logging "0.3.1"]]
```

```clj
(ns env-setup.core
  (:require [clojure.tools.logging :as log]))

(defn -main [& args]
  (log/info "Starting the app")
  (println "Hello, World!"))
```

Now when we run our app, we see some logs:

```bash
$ lein run
INFO: Starting the app
Hello World!
```

This isn't very convenient though, because we're only writing our logs to the console. It would be nicer to write to a file.
We can do that by using [log4j](http://logging.apache.org/log4j/1.2/).

### Log to a File

First, we'll need to add the log4j dependency to our ```project.clj```. We'll want to exclude the log4j dependencies that
we don't need:

```clj
:dependencies [[org.clojure/clojure "1.5.1"]
               [org.clojure/tools.logging "0.3.1"]
               [log4j/log4j "1.2.17" :exclusions [javax.mail/mail
                                                  javax.jms/jms
                                                  com.sun.jdmk/jmxtools
                                                  com.sun.jmx/jmxri]]]
```

Then, we'll need to add a ```log4j.properties``` file. I put this in the ```resources``` directory of my project.
The ```log4j.properties``` file configures our logger through log4j:

```
log4j.rootLogger=INFO, A1
log4j.appender.A1=org.apache.log4j.RollingFileAppender
log4j.appender.A1.File=log/app.log
log4j.appender.A1.MaxFileSize=500MB
log4j.appender.A1.MaxBackupIndex=2
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%d [%t] %-5p%c - %m%n
```

The first line of the property file above sets the ```rootLogger``` level to ```INFO``` and adds a logging
output destination, called an appender, named A1. Then the properties file configures our appender by making
it a [file appender](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/RollingFileAppender.html),
setting the file to log to, and the [logging pattern](http://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/PatternLayout.html)
for our logs.

Our logs now go to our ```app.log``` file and are formatted as we specified in the properties file:

```bash
$ lein run
Hello World!
```

```bash
$ cat log/app.log
2014-12-05 09:36:38,174 [main] INFO env-setup.core - Starting the app
```

### Use Environment Variables

To further improve our app environment, we'll probably want to use some environment variables. We can access
system environment variables through ```System/getenv```, which returns a map to reference:

```clj
(defn -main [& args]
  (let [env (get (System/getenv) "CLJ_ENV" "development")]
    (log/info (format "Starting the app with %s environment" env))
    (println "Hello, World!")))
```

So now our app will use our ```CLJ_ENV``` variable that we set when starting the app, or default to "development":

```bash
$ CLJ_ENV=production lein run
Hello World!
```

```bash
$ cat log/app.log
2014-12-05 09:36:38,174 [main] INFO env-setup.core - Starting the app with production environment
```

Obviously, we would want to use these environment for any type of configurations that would change from enviroment to
environment, like database setup, passwords, or remote service urls.

### Log to Environment Specific Log File

Back to solving the problem of our log files being in one place for all environments. It would be nice to have the logs
written to a file named for the environment. The problem with our current setup is that our ```log4j.properties``` file
can't access the system environment variables we are using in our app. The best way that I've found to inject
a variable to our log4j setup is through setting a jvm option in the lein profile. We can accomplish this pretty
easily by having a profile for each environment that sets the desired log file name through the jvm option:

```clj
:profiles {:dev        {:jvm-opts ["-Dlogfile.path=development"]}
           :test       {:jvm-opts ["-Dlogfile.path=test"]}
           :production {:jvm-opts ["-Dlogfile.path=production"]}}
```

Now we just change the file setting on the appender in the properties file to reference the ```logfile.path``` jvm option:

```
log4j.appender.A1.File=log/${logfile.path}.log
```

The logs now get written to the environment-specific file, which will default to whatever is specified in the ```:dev``` lein
profile.

```bash
$ lein run
Hello World!
```

```bash
$ cat log/development.log
2014-12-05 09:36:38,174 [main] INFO env-setup.core - Starting the app with development environment
```

Or, we can use Leiningen's ```with-profile``` to specify a profile to use:

```bash
$ lein with-profile production run
Hello World!
```

```bash
$ cat log/production.log
2014-12-05 09:36:38,174 [main] INFO env-setup.core - Starting the app with development environment
```

But, wait! You'll notice that we're no longer setting the system environment variable, so our log
in ```production.log``` shows that our app is still using the development environment. Rather than setting
environment settings through both jvm options and system environment variables, it would be nice
to unify these through Leiningen.

### Set Environment Variables in Profiles

We can do this pretty easily with [environ](https://github.com/weavejester/environ), which allows you to specify profile-specific
variables that would normally be set through system environment variables. Once we have the environ
dependency and plugin added to our ```project.clj```, we can set the ```:clj-env``` variable
in the environment-specific profiles that we already have:

```clj
:dependencies [[org.clojure/clojure "1.5.1"]
               [org.clojure/tools.logging "0.3.1"]
               [log4j/log4j "1.2.17" :exclusions [javax.mail/mail
                                                  javax.jms/jms
                                                  com.sun.jdmk/jmxtools
                                                  com.sun.jmx/jmxri]]
               [environ "1.0.0"]]
:plugins [[lein-environ "1.0.0"]]
:profiles {:dev        {:jvm-opts ["-Dlogfile.path=development"]
                        :env {:clj-env :development}}
           :test       {:jvm-opts ["-Dlogfile.path=test"]
                        :env {:clj-env :test}}
           :production {:jvm-opts ["-Dlogfile.path=production"]
                        :env {:clj-env :production}}}
```

Then, we require environ in our app, and use the ```:clj-env``` key, rather than the ```CLJ_ENV``` from ```System/getenv```:

```clj
(ns env-setup.core
  (:require [clojure.tools.logging :as log]
            [environ.core :as environ]))

(defn -main [& args]
  (let [env (environ/env :clj-env)]
    (log/info (format "Starting the app with %s environment" (name env)))
    (println "Hello, World!")))
```

Our lein profiles now configure the logs and the app environment from one place:

```bash
$ lein run
Hello World!
```

```bash
$ cat log/development.log
2014-12-05 09:36:38,174 [main] INFO env-setup.core - Starting the app with development environment
```

```bash
$ lein with-profile test run
Hello World!
```

```bash
$ cat log/test.log
2014-12-05 09:36:38,174 [main] INFO env-setup.core - Starting the app with test environment
```

### Use the Environment

The final step is to use the environment setup to run the app cleanly in different environments
and read environment-specific configuration. One nice way to do this would be to have ```lein test```
automatically use the test database. ```lein test``` will load the test profile by default, so we
just need to have a test database config to read. This could go in the ```:env``` map in the
lein profiles, or in a separate config file:

```clj
(ns env-setup.core
  (:require [clojure.tools.logging :as log]
            [environ.core :as environ]
            [env-setup.config :refer [config]]
            [env-setup.db :as db]))

(def db-connection
  (delay
    (let [env (environ/env :clj-env)]
      (-> config env :db db/connect!)
      (log/info (format "Using %s database" (name env))))))
```

```bash
$ lein test
```

```bash
$ cat log/test.log
2014-12-05 09:36:38,174 [main] INFO env-setup.core - Using test database
```

Logging and environment setup isn't the most interesting or fun task to pursue in your code,
but when it's done well it makes everything else a lot easier. There are many ways to configure
a Clojure app, but this method has worked well for me. I'd be interested to hear about other
or better ways that you manage environment setup through Leiningen. You can check out the entire
example app [here](https://github.com/kevbuchanan/clj-environment).
