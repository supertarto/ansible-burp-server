# Ansible Burp-server

[![CI](https://github.com/supertarto/ansible-burp-server/workflows/CI/badge.svg?event=push)](https://github.com/supertarto/ansible-burp-server/actions?query=workflow%3ACI)

Install and configure Burp backup Server with Ansible. This role is meant to suits my need and is inspired by https://github.com/CoffeeITWorks/ansible_burp2_server
If you want a more complete role, go check it!

## Tested plateform
* Debian 10 (Buster)

## Role variables
Boolean variables: Do we force the reinstallation of Burp? Do we enable the autoupgrade feature?
```yml
burp_force_reinstall: false
burp_server_autoupgrade_enabled: true
```
The path where the burp sources will be download and the format of the archive.
```yml
burp_download_dir: "{{ ansible_env.HOME }}/burp"

# .tar.gz, or zip
burp_srcext: "zip"
```
Burp Version.
```yml
# branch or tag github, For exemple: "master", "2.0.46"
burp_version: "2.2.18"
```
The .configure command line and some path used by Burp.
```yml
# Doc CFLAGS here: https://github.com/grke/burp/wiki/Performance-Tips#optional-compile-time-improvements
burp_configure_line: "./configure"
burp_usr_path: '/usr/local'
burp_bin_path: "{{ burp_usr_path }}/sbin/burp"

burp_src: "burp-{{ burp_version }}"
burp_url: "https://github.com/grke/burp/archive/{{ burp_version }}.{{ burp_srcext }}"


burp_server_etc: '/etc/burp'
burp_data_backup: /data_backup
burp_clientconf_path: "{{ burp_usr_path }}/burp"
burp_clientconf_path_dir: "{{ burp_clientconf_path }}/clientconfdir"
burp_clientconf_path_dir_incexc: "{{ burp_clientconf_path_dir }}/incexc"
burp_client_ca_csr_dir: "{{ burp_server_etc }}/CA-client"
```
You can describe here your "include" and "exclude" files and then assign those to your client. For exemple here, we create a template for windows7, one for windows 10, one for Debian as a server...
```yml

burp_incexc_templates:
  - name: includeW7
    include_regexp: []
    exclude_regexp: []

  - name: includeW10
    include_regexp: []
    exclude_regexp: []

  - name: includeDebianServer
    include_regexp: []
    exclude_regexp: []

  - name: includeDebianDesktop
    include_regexp: []
    exclude_regexp: []
```
Variables used to describe the "service" file.
```yml

# Service (systemd)
burp_sv_server_command: "{{ burp_bin_path }} -c {{ burp_server_etc }}/burp-server.conf -F"
burp_sv_server_user: "root"

burp_server_ca_conf: "{{ burp_server_etc }}/CA.cnf"
burp_server_address: "0.0.0.0"
burp_server_port: 4971
burp_server_status_port: 4972
```
Variables used in the burp-server.conf files. You must adapt those to suits your infrastructure needs.
```yml
# Conf file:
burp_server_listen: "{{ burp_server_address }}:{{ burp_server_port }}"
burp_server_listen_status: "127.0.0.1:{{ burp_server_status_port }}"
burp_server_dedup_group: global
burp_server_clientconfdir: "{{ burp_clientconf_path_dir }}"
burp_server_protocol: "1"
burp_server_pidfile: "/var/run/burp.server.pid"
burp_server_hardlinked_archive: "0"
# resume/delete
burp_server_working_dir_recovery_method: "delete"
burp_server_max_children: "5"
burp_server_max_status_children: "15"
burp_server_umask: "0027"
burp_server_syslog: "1"
burp_server_stdout: "0"
burp_server_client_can_delete: "0"
burp_server_client_can_force_backup: "1"
burp_server_client_can_list: "0"
burp_server_client_can_restore: "0"
burp_server_client_can_verify: "0"
burp_server_ratelimit: false
burp_server_network_timeout: false
burp_server_compression: zlib9
burp_server_version_warn: "1"
burp_server_autoupgrade_dir: "{{ burp_clientconf_path }}/autoupgrade/server"
burp_server_keep:
  - 7
  - 4
  - 3
burp_server_ca_name: myca-ca
burp_server_ca_server_name: myserver
burp_server_ca_burp_ca: "{{ burp_usr_path }}/sbin/burp_ca"
burp_server_ca_crl_check: "1"
burp_server_ssl_cert_ca: "{{ burp_server_etc }}/ssl_cert_ca-server.pem"
burp_server_ssl_cert: "{{ burp_server_etc }}/ssl_cert-server.pem"
burp_server_ssl_key: "{{ burp_server_etc }}/ssl_cert-server.key"
burp_server_ssl_key_password: "ChangeIT!"
burp_server_ssl_dhfile: "{{ burp_server_etc }}/dhfile.pem"
burp_server_timer_script: "{{ burp_usr_path }}/share/burp/scripts/timer_script"
burp_server_timer_args:
  - 20h
  - Mon,Tue,Wed,Thu,Fri,Sat,00,01,02,03,04,05,06,07,08,09,10,11,12,13,14,15,16,17,18,19,20,21,22,23
burp_notify_failure: true
burp_notify_failure_email_to: "myemail@localhost.com"
burp_notify_failure_email_from: "Burp_Backup"
burp_restore_super_clients:
  - monitor
```
A list of client to deploy. You must define a password and you can use a profile, among those you have created with **burp_incexc_templates**
```yml
burp_add_clients: []
#  - name: client1
#    profile: includeW10
#    password: password
```
A list of client to remove. The certs and csr will be removed
```yml
burp_remove_clients: []
# - name: client40
```
Auto upgrade configuration
```yml
burp_autoupgrade_version: "2.2.18"
burp_autoupgrade_package_copy:
  - src: "https://sourceforge.net/projects/burp/files/burp-{{ burp_autoupgrade_version }}/burp-win64-installer-{{ burp_autoupgrade_version }}.exe/download"
    dest: "{{ burp_server_autoupgrade_dir }}/win64/{{ burp_autoupgrade_version }}/package"
```
Local client, use for restoration purpose.
```yml
burp_client_pidfile: "/var/run/burp.pid"
burp_client_password: "changeIT!"
burp_client_ssl_cert_ca: "{{ burp_server_etc }}/ssl_cert_ca-monitor.pem"
burp_client_ssl_cert: "{{ burp_server_etc }}/ssl_cert-monitor.pem"
burp_client_ssl_key: "{{ burp_server_etc }}/ssl_cert-monitor.key"
burp_client_ssl_peer_cn: "{{ ansible_hostname }}"
```

## Installation
```
ansible-galaxy install supertarto.burp_server
```
## License
GPL V3.0
