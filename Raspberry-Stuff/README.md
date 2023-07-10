# Unordered stuff

- Connect headless PI to wifi
  - `raspi-config` (needs a reboot after, currently only IPV4)
- Can't configure PiW for remote VSC :(
- Use `raspi-config` to turn on SSH
- Splunk create events
  - In Postman under splunk-hetzner
  - Note that curl needs a `-k` option if Splunk is on self signed certificates
- Setting up Splunk
  - Need to renew TF/Ansible/cloud-init stuff for V9



# Miscellaneous

## Download Splunk wget

``` bash
wget -O splunk-9.1.0.1-77f73c9edb85-Linux-x86_64.tgz "https://download.splunk.com/products/splunk/releases/9.1.0.1/linux/splunk-9.1.0.1-77f73c9edb85-Linux-x86_64.tgz"
```
