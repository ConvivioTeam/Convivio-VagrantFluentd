# Convivio Fluentd Vagrant box

This is a [Vagrant](https://www.vagrantup.com/) box that creates a [Fluentd](https://www.fluentd.org/) instance for local development.

> **Fluentd is an open source data collector for unified logging layer.**
> Fluentd allows you to unify data collection and consumption for a better use and understanding of data.
> https://www.fluentd.org/

#### Requirements:

* Vagrant >= 1.8.6
* Ansible >= 2.4

**HT:** [Drupal VM](https://www.drupalvm.com/)
The structure and functioning of this Vagrant box draws inspiration and more from the _Drupal VM_ Vagrant box project.

## What it does

This Vagrant box installs Fluentd on a Vagrant VM (by default, Ubuntu Xenial (16.04LTS)). It uses the [jebovic/ansible-fluentd](https://github.com/jebovic/ansible-fluentd) Ansible role by [Jérémy Baumgarth](https://github.com/jebovic). 

The box also installs Elasticsearch with the [elastic/ansible-elasticsearch](https://github.com/elastic/ansible-elasticsearch) Ansible role from [Elastic](https://www.elastic.co/) so that out-of-the-box Fluentd can use Elasticsearch as an output. 

This box includes default configuration that should make sense in most circumstances. However, the configuration is designed to allow every setting to be overridden, according to your needs and circumstances.

## TODO

1. Improve the default configuration.
2. Document the default configuration properly.
2. Make this work on Docker as well as Vagrant.