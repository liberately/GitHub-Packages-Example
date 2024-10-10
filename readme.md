# Publish and Use a Private Android Library to GitHub Packages

This guide will walk you through uploading your private Android library to GitHub Packages and using it in other projects. Ensure you have a GitHub Token ready to authenticate with the private repository.

## Prerequisites

Create a `local.properties` file in the root directory of your project to store GitHub authentication information:

```properties
GITHUB_URL=https://maven.pkg.github.com/{your_github_username}/{your_repository_name}
GITHUB_USERNAME=your_github_username
GITHUB_TOKEN=your_github_token
```

Make sure **not to commit this file to version control**.

## Publishing a Private Library to GitHub Packages

Add the following configuration to the `build.gradle.kts` file in your library project:

### Module's `build.gradle.kts` File

```kotlin
import java.io.FileInputStream
import java.util.Properties

plugins {
    id("maven-publish")
}

tasks.withType<JavaCompile> {
    options.encoding = "UTF-8"
}
val prop = Properties().apply {
    load(FileInputStream(File(rootProject.rootDir, "local.properties")))
}
afterEvaluate {
    publishing {
        repositories {
            maven {
                name = "GitHub-Packages-Example"
                setUrl(prop.getProperty("GITHUB_URL"))
                credentials {
                    username = prop.getProperty("GITHUB_USERNAME")
                    password = prop.getProperty("GITHUB_TOKEN")
                }
            }
        }
        publications {
            create<MavenPublication>("maven") {
                from(components["release"])
                groupId = "com.example"
                artifactId = "library"
                version = "1.0.0"
            }
        }
    }
}
```

### Publishing the Library

Run the following command in the project directory to publish the library to GitHub Packages:

```bash
./gradlew publish
```

## Using the Private Library in Another Project

To use this published library in another project, configure the GitHub repository and dependency in the `build.gradle.kts` file of that project.

### Root Project's `build.gradle.kts` File

```kotlin
import java.io.FileInputStream
import java.util.Properties

val prop = Properties().apply {
    load(FileInputStream(File(rootProject.projectDir, "local.properties")))
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven {
            setUrl(prop.getProperty("GITHUB_URL"))
            credentials {
                username = prop.getProperty("GITHUB_USERNAME")
                password = prop.getProperty("GITHUB_TOKEN")
            }
        }
    }
}

```

### Project's `build.gradle.kts` File

Add the library dependency to the module where you want to use the library:

```kotlin
dependencies {
    implementation("com.example:library:1.0.0")
}
```

## Common Issues

### Unable to Access the Private Library

If access to the private library fails, check the following:
- Ensure the GitHub Token has **`repo` and `packages`** permissions.
- Ensure the credentials in the `local.properties` file are correct.

### Publishing Failed

If publishing fails, it is usually due to authentication issues or a configuration problem with the `local.properties` file. Ensure your `local.properties` file is present and correctly loaded.

Following this guide, you should be able to successfully upload an Android library to GitHub Packages and use it in other projects. For further assistance, consult the [GitHub documentation](https://docs.github.com/en/packages) or GitHub Discussions.