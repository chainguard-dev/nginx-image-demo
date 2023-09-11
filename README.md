# nginx-image-demo

This is a demonstration of how [cosign][cs], [apko][apko] and [Melange][mel]
can be combined to build an image with a custom application payload.  It uses
a set of GitHub Actions published in the Chainguard [actions][actions] repository.

   [cs]: https://github.com/sigstore/cosign
   [apko]: https://github.com/chainguard-dev/apko
   [mel]: https://github.com/chainguard-dev/melange
   [actions]: https://github.com/chainguard-dev/actions

## The lifecycle of an application build

### Packaging

First, the relevant packages to support an application are built with Melange!
In this case, we are using nginx as an example (a stripped down version of the
nginx ingress packaging, which builds a few dozen dependencies).

You can see an example of this in the `.melange.yaml` file.

### Image composition and publish

Next, the image is composed and published with `apko`.  Melange stores its packages
in a local repository, which is consumed by `apko`.  These packages are combined
with dependencies from the upstream Alpine Linux distribution, composed into an
OCI image, and published.  An SBOM is generated and published along side the
image if `apko` 0.3 or newer is used.

Apko configures an s6 service bundle and arranges for it to be launched when a
container is started.  This is similar to using `s6-overlay` with Docker, and is
considered a best practice so that zombie processes get reaped.

### Image signing

Finally, the image is signed using Cosign.  You can see the `.github/workflows/push.yaml`
file for the details on how this works.

All of this is done in a declarative (and reproducible) way.  Since it is declarative
and reproducible, refreshing the image can be automated.  See the [distroless][dl]
GitHub project for some examples of how this can be done.

   [dl]: https://github.com/distroless
