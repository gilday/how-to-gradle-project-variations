# how-to-gradle-project-variations

Gradle example for generating dynamic subprojects with variations of the
dependency graph.

# Building

    ./gradlew war

# Why

Suppose one needs to build a test web application for testing compatibility with
an external tool such as a Java agent. It would be helpful to know that this
test web application works with various versions of the web framework upon which
it was built, so we want to repeat the project build with several versions of
the web framework. The simplest way to do this is to introduce a Gradle property
to represent the version of the web framework and use this property when
defining the project's dependencies. Next, one can simply repeat the build with
a shell script.

```bash
for version in 4.3.12 4.3.13 4.3.14; do
  ./gradlew war -PspringVersion=$version
done
```

# Proposed Solution: dynamic subprojects

Use settings.gradle to dynamicaly generate subprojects for each of the
variations of the web application we want to build.

    | how-to-gradle-project-variations
    ├── app
    ├── build.gradle
    ├── settings.gradle
    └── variations

The "app" directory contains the web application project

    app/
    └── src
        └── main
            └── java
                └── com
                    └── johnathangilday
                        └── HelloWorldServlet.java

settings.gradle generates a subproject for each of the variations of the
servlet-api dependency with which we want to build this project

```groovy
// define versions of javax.servlet:servlet-api for which to generate variations
of the app
def variations = ['2.3', '2.4', '2.5']

def projects = variations.collect { "${rootProject.name}-${it}" }

// include the app subproject with the servlet definitions
include('app')
// include a subproject for each variation of the servlet-api
include(*projects)
```

`/.gradlew projects` shows us the dynamic subprojects generated in the
"variations" directory

    $ ./gradlew projects

    > Task :projects

    ------------------------------------------------------------
    Root project
    ------------------------------------------------------------

    Root project 'how-to-gradle-project-variations'
    +--- Project ':app'
    +--- Project ':how-to-gradle-project-variations-2.3'
    +--- Project ':how-to-gradle-project-variations-2.4'
    \--- Project ':how-to-gradle-project-variations-2.5'
