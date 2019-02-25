# Docker Style Guide

### Use exec form of CMD

Don't do:
```
FROM ubuntu
CMD wc --help
```

Do this:
```
FROM ubuntu
CMD ["/usr/bin/wc","--help"]
```

[Docs](https://docs.docker.com/engine/reference/builder/#cmd) state:

> This array form is the preferred format of CMD