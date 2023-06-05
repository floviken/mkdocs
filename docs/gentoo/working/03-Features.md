### Distributing prebuilt packages to others

If prebuilt packages are to be distributed to others, then make sure that this is permitted. Check the distribution terms of the upstream package for this. For example, for a package released under the GNU GPL, sources must be made available along with the binaries.

Ebuilds may define a `bindist` restriction in their *RESTRICT* variable if built binaries are not distributable. Sometimes this restriction is conditional on one or more USE flags.

By default, Portage will not mask any packages because of restrictions. This can be changed globally by setting the *ACCEPT_RESTRICT* variable in /etc/portage/make.conf. For example, to mask packages that have a `bindist` restriction, add the following line to make.conf:

``` sh title="FILE /etc/portage/make.confOnly accept binary distributable packages"
ACCEPT_RESTRICT="* -bindist"
```

It is also possible to override the *ACCEPT_RESTRICT* variable by passing the `--accept-restrict` option to the **emerge** command. For example, `--accept-restrict=-bindist` will temporarily mask packages with a `bindist` restriction.

Also consider setting the *ACCEPT_LICENSE* variable when distributing packages. See the Licenses section for this.

!!! Important

It is entirely the responsibility of each user to comply with packages' license terms and with laws of each user's country. The metadata variables defined by ebuilds (*RESTRICT* or *LICENSE*) can provide guidance when distribution of binaries is not permitted, however output from Portage or questions answered by the Gentoo developers are not legal statements and should not be relied upon as such. Be cautious to abide by the law of your physical location.

## Fetching files

### Userfetch

Portage is normally run as the root user. Setting `FEATURES="userfetch"` will allow Portage to drop root privileges while fetching package sources and run with user/group permissions of portage:portage. This is a small security improvement.

If `userfetch` is set in *FEATURES* be sure to change the owner of all the files beneath /var/db/repos/gentoo using the **chown** command with root privileges:

```sh 
root # chown --recursive --verbose portage:portage /var/db/repos/gentoo
```

### Verify distfiles

To re-verify the integrity and (potentially) re-download previously removed/corrupted distfiles for all currently installed packages, run:
``` sh
root # emerge --ask --fetchonly --emptytree @world
``` 
