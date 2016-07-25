# gradle-api-impl-separation
One approach to enforce clear interfaces between different modules, is to separate the API from the implementation and allow other modules to only depend on API artifacts.
For example: 
If `moudleB` depends on `moudleA`, then `moduleB` is only allowed to reference classes in the API part of `moduleA`.

This repository demonstrates a gradle multiproject build configuration that achieves this separation using [source sets](https://docs.gradle.org/2.14.1/dsl/org.gradle.api.tasks.SourceSet.html) and [dependency configurations](https://docs.gradle.org/2.14.1/dsl/org.gradle.api.Project.html#org.gradle.api.Project:configurations(groovy.lang.Closure)).

Per module there are two source sets defined:
- main: This source set contains the implementation classes (it points to src/impl/java)
- api: This source set contains the api classes that should be exposed

```
sourceSets {
    main {
        java {
            srcDirs = ['src/impl/java']
        }

        resources {
            srcDir = ['src/impl/resources']
        }
    }

    api
}
```

For each source there is a corresponding configuration that has a JAR artifact only containing the compiled sources from the source set.
This means that for each module a `module-api.jar` contianing only the API classes and a `module.jar` containing only the implementation classes is created.

```

task apiJar(type: Jar) {
    baseName "${project.name}-api"
    from sourceSets.api.output
}

artifacts {
    apiCompile apiJar
}
```

By default the implementation/main configuration of a module always has a dependency to it's api configuration.
```
dependencies {
    compile project(path: ":${project.name}", configuration: "apiCompile")
}
```

A dependency from `moduleA` to `moduleB`s API can then be defined as follows:
```
dependencies {
    compile project(path: ":moduleA", configuration: "apiCompile")
}
```
