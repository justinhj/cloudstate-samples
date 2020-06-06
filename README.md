
# Cloudstate Sample Applications

## Prerequisites

The following assumes that you have completed the steps for setting up your local environment as well as creating an account and project.  If you have not done this you must follow the instructions here:

* [Setting Up your Machine](https://docs.lbcs.dev/gettingstarted/setup.html)
   * as well as the [Developer prerequisites](https://docs.lbcs.dev/developing/developing.html#prerequisites)
   * Install [nvm](https://github.com/nvm-sh/nvm#install--update-script) (node version manager)
      * We recommend v0.34.0 or later.  (Check with `nvm --version`)
   * Install [npm](https://www.npmjs.com/get-npm) (node package manager)
      * We recommend v6.14.3 or later.  (Check with `npm -v`)
   * You also need to install the protobuf compiler.
      * We recommend using v3.0.0 or later.  (Check with `protoc --version`)
      * Mac OS X `brew install protobuf`
      * Linux `sudo apt install protobuf-compiler`
      * Or [alternatively](https://developers.google.com/protocol-buffers/docs/downloads) (src and bins)
* [Your Lightbend Cloudstate Account](https://docs.lbcs.dev/gettingstarted/account.html)
* [Creating a Project](https://docs.lbcs.dev/gettingstarted/project.html)

## Sample application layout
Grab the sample application from github:
 
[https://github.com/lightbend/cloudstate-samples](https://github.com/lightbend/cloudstate-samples)

## Shopping Cart
Event sourced example with PostgreSQL `statefulstore`.  Simple shop front end to interact with.
* [shopping-cart](shopping-cart)

## Chat sample
CRDT chat with simple front end web interface, friends storage, and presence state.
* [chat](chat)

## Sample event sourced application
Event sourced example with PostgreSQL `statefulstore`.  This example provides a back-end service that implements a shopping cart like functionality, allowing for requests to be send against it in order to store, retrieve and delete data.
* [eventsource](eventsource)

## Dev notes

### Nodes.js version
We pick up node.js version 12 as default in all node.js related projects. 
Based on [node.js website](https://nodejs.org/en/), currently version 12 is recommended for most users. 
Based on [node.js release page](https://nodejs.org/en/about/releases/), the odd number release has only 6 months support cycle, so we don't set it as default.
If you plan to try the different node.js version in sample projects, you can
 - modify `.nvmrc` file, so command `nvm install` and `nvm use` will pick up the node.js version you set.
 - modify `package.json` file, especially "engineStrict" and "engines.node" fields. It affects the version check when running `npm install`

## Maintenance notes

### License
The license is Apache 2.0, see [LICENSE-2.0.txt](LICENSE-2.0.txt).

### Maintained by
__This project is NOT supported under the Lightbend subscription.__

This project is maintained mostly by @coreyauger and @cloudstateio.

Feel free to ping above maintainers for code review or discussions. Pull requests are very welcome–thanks in advance!


### Disclaimer

[DISCLAIMER.txt](DISCLAIMER.txt)
