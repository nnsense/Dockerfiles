## Image content

This the same image as `ci_tools_alpine` but it also include `mvn 3.6.3`

`python:rc-alpine3.13` based image for CI automation featuring the following packages on top of the base:
- python3
- jq
- bash
- git
- openssh-client
- curl
- binutils
- xmlstarlet

For CI automation the following tools have been added:
- jfrog (lateest)
- helm (latest)
- terraform (0.13.6)
- aws cli (latest v2)
- kubectl (latest)
- mvn (3.6.3)