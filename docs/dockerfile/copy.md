This section is about another Dockerfile keyword: COPY.

So far, we have installed things in our container images by downloading packages.

We can also copy files from the build context to the container that we are building.

Remember: the build context is the directory containing the Dockerfile.


### Build some C code

We want to build a container that compiles a basic "Hello world" program in C.

Let's use a simp hello.c:

```c
int main () {
  puts("Hello, world!");
  return 0;
}
```
Let's create a new directory, and put this file in there.

Then we will write the Dockerfile.

On Debian and Ubuntu, the package build-essential will get us a compiler.

When installing it, don't forget to specify the -y flag, otherwise the build will fail (since the build cannot be interactive).

Then we will use COPY to place the source file into the container

```Dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install -y build-essential
COPY hello.c /
RUN make hello
CMD /hello
```

**Exercise:**
Create hello.c and Dockerfile in the same directory.

Run docker build -t hello . in this directory.

Run docker run hello, you should see Hello, world!.

### COPY and the build cache

Run the build again.

- Now, modify hello.c and run the build again

 Docker can cache steps involving COPY. Those steps will not be executed again if the files haven't been changed.

### The .dockerignore file

We can create a file named .dockerignore

(at the top-level of the build context)

It can contain file names and globs to ignore

They won't be sent to the builder

(and won't end up in the resulting image)

See the [documentation](https://docs.docker.com/engine/reference/builder/#dockerignore-file) for the little details



