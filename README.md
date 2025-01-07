# Reproducer for [MNG-8486](https://issues.apache.org/jira/browse/MNG-8486)

This example requires a local Docker daemon to be running.
I've used [Rancher Desktop](https://rancherdesktop.io) for this reproducer myself, but it shouldn't matter which one you use.

## Successful flow with Maven 3.9.9

Compile the code and let Jib build a container image to your local Docker daemon and provide the name via a property on the command line (as shown [here](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin#system-properties) in the Jib documentation):

    ❯ mvn compile jib:dockerBuild -Djib.to-image=foo:bar
    (...)
    [INFO] Built image to Docker daemon as foo:bar
    (...)

You should now have a `foo:bar` image:

    ❯ docker images | grep foo
    foo              bar              36025ca8871e   55 years ago   287MB

Feel free to remove it again:

    ❯ docker rmi foo:bar

## Unsuccessful flow with Maven 4.0.0-rc-2

Compile the code and let Jib build a container image to your local Docker daemon and provide the name via a property on the command line (as shown [here](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin#system-properties) in the Jib documentation):

    ❯ mvn compile jib:dockerBuild -Djib.to-image=foo:bar
    (...)
    [INFO] Built image to Docker daemon as example:1.0.0-SNAPSHOT
    (...)

It seems that the `jib.to.image` property is not picked up, and the image gets a default name based on the project's `artifactId` and `version`.
However, I would have expected to end up with a `foo:bar` image instead of an `example:1.0.0-SNAPSHOT` image, just like with Maven 3.9.9.

To confirm, you will now not have a `foo:bar` image:

    ❯ docker images | grep -c foo                        
    0

And you will indeed have an `example:1.0.0-SNAPSHOT` image:

    ❯ docker images | grep example
    example          1.0.0-SNAPSHOT   36025ca8871e   55 years ago   287MB

Feel free to remove it again:

    ❯ docker rmi example:1.0.0-SNAPSHOT