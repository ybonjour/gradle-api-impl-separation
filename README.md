# gradle-api-impl-separation
One approach to have clear interfaces amongst different modules, is to separate the API from the implementation for each module.
Modules are then only allowed to reference API classes of other modules.
For example: 
If `moudleB` depends on `moudleA`, then `moduleB` is only allowed to reference classes in the API part of `moduleA`.

This repository demonstrates a gradle multiproject build configuration that allows to separate API and implementation classes based on source sets and configurations.

Per module there are two source sets defined:
- main: This source set contains the implementation classes (it points to src/impl/java)
- api: This source set contains the api classes that should be exposed

```
sourceSets {
    main {
        java {
            srcDir 'src/impl/java'
        }

        resources {
            srcDir 'src/impl/resources'
        }
    }

    api {
        java {
            srcDir 'src/api/java'
        }
    }
}
```

For each source there is a corresponding configuration with a JAR artifact only containing the compiled sources from the source set.
This means that for each module a `module-api.jar` contianing only the API classes and a `module.jar` containing only the implementation classes is created.

```
configurations {
    api
}

task apiJar(type: Jar) {
    baseName "${project.name}-api"
    from sourceSets.api.output
}

artifacts {
    api apiJar
}
```

By default a implementation/main configuration always has a dependency to it's api configuration.
```
dependencies {
    compile project(path: ":${project.name}", configuration: "api")
}
```

A dependency from `moduleA` to `moduleB`s API can then be defined as follows:
```
dependencies {
    compile project(path: ":moduleA", configuration: "api")
}
```
