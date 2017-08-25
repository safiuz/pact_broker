# Pact Broker
[![Gem Version](https://badge.fury.io/rb/pact_broker.svg)](http://badge.fury.io/rb/pact_broker)
 [![Build Status](https://travis-ci.org/pact-foundation/pact_broker.svg?branch=master)](https://travis-ci.org/pact-foundation/pact_broker) [![Join the chat at https://gitter.im/pact-foundation/pact_broker](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/pact-foundation/pact_broker?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
 [![Code Climate](https://codeclimate.com/github/pact-foundation/pact_broker/badges/gpa.svg)](https://codeclimate.com/github/pact-foundation/pact_broker)
 [![Test Coverage](https://codeclimate.com/github/pact-foundation/pact_broker/badges/coverage.svg)](https://codeclimate.com/github/pact-foundation/pact_broker/coverage)
 [![Issue Count](https://codeclimate.com/github/pact-foundation/pact_broker/badges/issue_count.svg)](https://codeclimate.com/github/pact-foundation/pact_broker)
 [![Dependency Status](https://gemnasium.com/badges/github.com/pact-foundation/pact_broker.svg)](https://gemnasium.com/github.com/pact-foundation/pact_broker)

The Pact Broker provides a repository for consumer driven contracts created using the pact gem.

It:

* solves the problem of how to share pacts between consumer and provider projects
* allows you to [decouple your service release cycles][decouple]
* provides API documentation that is guaranteed to be up-to date
* shows you real examples of how your services interact
* allows you to visualise the relationships between your services

Features:

* A RESTful API for publishing and retrieving pacts.
* An embedded HAL browser for navigating the API.
* Autogenerated documentation for each pact.
* Dynamically generated network diagrams so you can visualise your microservice network.
* Displays provider verificaton results so you know if you can deploy safely.
* Provides badges to display pact verification statuses in your READMEs.
* Enables a pact version to be tagged (ie. "prod") so a provider can verify itself against a fixed version of a pact to ensure backwards compatibility.
* Provides webhooks to trigger a provider build when a consumer publishes a change to a pact.
* Tracks changes between Pact versions so you can tell when a consumer has changed its expectations.
* [Docker Pact Broker][docker]

### How would I use the Pact Broker?

#### Step 1. Consumer CI build
1. The consumer project runs its tests using the [Pact][pact] library to provide a mock service.
2. While the tests run, the mock service writes the requests and the expected responses to a JSON "pact" file - this is the consumer contract.
3. The generated pact is then published to the Pact Broker. This is simply a `PUT` to a resource that specifies the consumer name and version, and the provider name. eg `http://my-pact-broker/pacts/provider/Animal%20Service/consumer/Zoo%20App/version/1.0.0`
4. When a pact is published, a webhook in the Pact Broker kicks off a build of the provider project if the pact content has changed since the previous version.

#### Step 2. Provider CI build
1. The provider has a verification task that is configured with the URL to retrieve the latest pact between itself and the consumer. eg `http://my-pact-broker/pacts/provider/Animal%20Service/consumer/Zoo%20App/latest`.
2. The provider build runs the pact verification task, which retrieves the pact from the Pact Broker, replays each request against the provider, and checks that the responses match the expected responses.
3. If the pact verification fails, the build fails. The [Pact Broker CI Nerf Gun][nerf] magically determines who caused the verification to fail, and shoots them.
4. The results of the verification are published back to the Pact Broker by the pact verification tool, so the consumer team will know if the code they have written will work IRL.

If you don't have a [Pact Broker CI Nerf Gun][nerf], you'll probably want to read about using pact when the consumer and provider are being written by [different teams][different-teams].

## Documentation

See the [wiki][wiki] for documentation on the Pact Broker.

## Support

* Check the [wiki][wiki] first.
* See if there is an existing or closed [issue][issues] and raise a new issue if not.
* See if there is an existing question on [stackoverflow][stackoverflow] tagged with `pact-broker`, and ask a new question if not.
* Have a chat to us on the Pact [gitter][gitter].
* Tweet us at [@pact_up][twitter] on the twitters.

### Screenshots

#### Index

* * *
<img src="https://raw.githubusercontent.com/wiki/pact-foundation/pact_broker/images/index.png"/>

#### Autogenerated documentation

Paste the pact URL into a browser to view a HTML version of the pact.
* * *
<img src="https://raw.githubusercontent.com/wiki/pact-foundation/pact_broker/images/autogenerated_documentation.png"/>


#### Network diagram

* * *
<img src="https://raw.githubusercontent.com/wiki/pact-foundation/pact_broker/images/network_diagram.png"/>

#### HAL browser

Use the embedded HAL browser to navigate the API.
* * *
<img src="https://raw.githubusercontent.com/wiki/pact-foundation/pact_broker/images/hal_browser.png"/>

#### HAL documentation

Use the HAL browser to view documentation as you browse.
* * *
<img src="https://raw.githubusercontent.com/wiki/pact-foundation/pact_broker/images/hal_documentation.png"/>

## Usage

### To have a play around on your local machine

* Install ruby 2.2.0 or later and bundler >= 1.12.0
    * Windows users: get a Rails/Ruby installer from [RailsInstaller](http://railsinstaller.org/) and run it
    * unix users just use their package manager
* Run `git clone https://github.com:pact-foundation/pact_broker.git && cd pact_broker/example`
* Run `bundle install`
* Run `bundle exec rackup -p 8080`
* Open [http://localhost:8080](http://localhost:8080) and you should see a list containing the pact between the Zoo App and the Animal Service.
* Click on the arrow to see the generated HTML documentation.
* Click on either service to see an autogenerated network diagram.
* Click on the HAL Browser link to have a poke around the API.
* Click on the book icon under "docs" to view documentation related to a given relation.

### For reals

#### Hosted

In a hurry? Hate having to run your own infrastructure? Check out the [Hosted Pact Broker][hosted] - it's fast, it's secure and it's free!

#### Container solutions

You can use the [Pact Broker Docker container][docker] or [Terraform on AWS][terraform]

#### Rolling your own

* Create a PostgreSQL (recommended) or MySQL (not recommended, see following note) database.
* __Note:__ It is recommended to use __PostgreSQL__ as it will support JSON search features that are planned in the future, however MySQL the other [semi supported](https://github.com/pact-foundation/pact_broker/issues/33) database.
* Install ruby 2.2.0 or later and bundler >= 1.12.0
* Copy the [example](/example) directory to the location you want to install the application.
* Modify the config.ru and Gemfile as desired (eg. choose database driver gem, set your database credentials. Use the "pg" gem if using Postgres and the "mysql2" gem if using MySQL)
* Please ensure you use `encoding: 'utf8'` in your Sequel options to avoid encoding issues.
* For production usage, use a web application server like [Phusion Passenger](https://www.phusionpassenger.com) or [Nginx](http://nginx.org/) to serve the Pact Broker application.
* Deploy to your location of choice.

[decouple]: http://techblog.realestate.com.au/enter-the-pact-matrix-or-how-to-decouple-the-release-cycles-of-your-microservices/
[pact]: https://github.com/realestate-com-au/pact
[nerf]: https://github.com/pact-foundation/pact_broker/wiki/pact-broker-ci-nerf-gun
[different-teams]: https://github.com/realestate-com-au/pact/wiki/Using-pact-where-the-consumer-team-is-different-from-the-provider-team
[docker]: https://hub.docker.com/r/dius/pact-broker
[terraform]: https://github.com/nadnerb/terraform-pact-broker
[hosted]: https://pact.dius.com.au/?utm_source=github&utm_campaign=GITHUB_BROKER&utm_medium=github
[wiki]: https://github.com/pact-foundation/pact_broker/wiki
[stackoverflow]: http://stackoverflow.com/questions/tagged/pact-broker
[twitter]: https://twitter.com/pact_up
[gitter]: https://gitter.im/realestate-com-au/pact
[issues]: https://github.com/pact-foundation/pact_broker/issues
