## Linux-Unix - SELinux

By Jack Szwergold

### The basics.

Check the SELinux status with this command:

    sestatus

Check the status of—for example—HTTPD related services like this:

	getsebool -a | grep httpd

The output should be something like this:

	httpd_anon_write --> off
	httpd_builtin_scripting --> on
	httpd_can_check_spam --> off
	httpd_can_connect_ftp --> off
	httpd_can_connect_ldap --> off
	httpd_can_connect_mythtv --> off
	httpd_can_connect_zabbix --> off
	httpd_can_network_connect --> off
	httpd_can_network_connect_cobbler --> off
	httpd_can_network_connect_db --> off
	httpd_can_network_memcache --> off
	httpd_can_network_relay --> off
	httpd_can_sendmail --> off
	httpd_dbus_avahi --> off
	httpd_dbus_sssd --> off
	httpd_dontaudit_search_dirs --> off
	httpd_enable_cgi --> on
	httpd_enable_ftp_server --> off
	httpd_enable_homedirs --> off
	httpd_execmem --> off
	httpd_graceful_shutdown --> on
	httpd_manage_ipa --> off
	httpd_mod_auth_ntlm_winbind --> off
	httpd_mod_auth_pam --> off
	httpd_read_user_content --> off
	httpd_run_ipa --> off
	httpd_run_preupgrade --> off
	httpd_run_stickshift --> off
	httpd_serve_cobbler_files --> off
	httpd_setrlimit --> off
	httpd_ssi_exec --> off
	httpd_sys_script_anon_write --> off
	httpd_tmp_exec --> off
	httpd_tty_comm --> off
	httpd_unified --> off
	httpd_use_cifs --> off
	httpd_use_fusefs --> off
	httpd_use_gpg --> off
	httpd_use_nfs --> off
	httpd_use_openstack --> off
	httpd_use_sasl --> off
	httpd_verify_dns --> off

So if we wanted to enable HTTPD networking for DB connections we would run this command:

	setsebool httpd_can_network_connect_db 1

Or this variant of the command to make the change permanent:

	sudo setsebool -P httpd_can_network_connect_db 1

***

To do a similar thing for LDAP connections by running this command:

	getsebool -a | grep ldap

The output should be something like this:

	authlogin_nsswitch_use_ldap --> off
	dhcpd_use_ldap --> off
	httpd_can_connect_ldap --> off

And to enable HTTPD networking for LDAP connections we would run this command:

	setsebool httpd_can_connect_ldap 1

Or this variant of the command to make the change permanent:

	sudo setsebool -P httpd_can_connect_ldap 1

### SELinux contexts, files and directories.

If there are directory related issues — tied to reading and/or writing in general or to directories outside of `/var/www/html/` — you can run this `chcon` command but the changes will be temporary:

	chcon -vR -t httpd_sys_rw_content_t /opt/
	chcon -vR -t httpd_sys_rw_content_t /var/www/html/

You must install `semanage` since it is not installed by default. Search for what package installs it:

	yum whatprovides /usr/sbin/semanage

And it should show `policycoreutils-python`:

	yum install policycoreutils-python

Then run this `semanage` command to make these commands permanent:

	semanage fcontext -a -t httpd_sys_rw_content_t "/opt(/.*)?"
	semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html(/.*)?"

You can then check the status by using `ls` with the `-lZ` parameters:

	ls -latrZ /opt/
	ls -latrZ /var/www/html/

And if — for some reason — you need to restore original values, do this:

	restorecon -vR /opt/
	restorecon -vR /var/www/html/

Test stuff:

Create a test file like this:

	touch /opt/something.txt
	chown apache:apache /opt/something.txt
	ls -lZ /opt/something.txt

***

*Linux-Unix - SELinux (c) by Jack Szwergold; written on August 7, 2017. This work is licensed under a Creative Commons Attribution-NonCommercial 4.0 International License (CC-BY-NC-4.0).*
