#!/usr/bin/env bash
#
################################################################################
#  duply (grown out of ftplicity), is a shell front end to duplicity that      #
#  simplifies the usage by managing settings for backup jobs in profiles.      #
#  It supports executing multiple commands in a batch mode to enable single    #
#  line cron entries and executes pre/post backup scripts.                     #
#  Since version 1.5.0 all duplicity backends are supported. Hence the name    #
#  changed from ftplicity to duply.                                            #
#  See http://duply.net or http://ftplicity.sourceforge.net/ for more info.    #
#  (c) 2006 Christiane Ruetten, Heise Zeitschriften Verlag, Germany            #
#  (c) 2008-2023 Edgar Soldin (changes since version 1.3)                      #
################################################################################
#  LICENSE:                                                                    #
#  This program is licensed under GPLv2.                                       #
#  Please read the accompanying license information in gpl-2.0.txt.            #
################################################################################
#  TODO/IDEAS/KNOWN PROBLEMS:
#  - possibility to restore time frames (incl. deleted files)
#    realizable by listing each backup and restore from 
#    oldest to the newest, problem: not performant
#  - search file in all backups function and show available
#    versions with backups date (list old avail since 0.6.06)
#  - edit profile opens conf file in vi 
#  - implement log-fd interpretation
#  - add a duplicity option check against the options pending 
#    deprecation since 0.5.10 namely --time-separator
#                                    --short-filenames
#                                    --old-filenames
#  - add 'exclude_<command>' list usage e.g. exclude_verify
#  - featreq 25: a download/install duplicity option
#  - import/export profile from/to .tgz function !!!
#  - remove url_encode, test for invalid chars n throw error instead
#
#  CHANGELOG:
#  2.5.2 (30.11.2023)
#  - bug #139: "ampersand (&) in gpg passphrase breaks gpg tests"
#
#  2.5.1 (4.10.2023)
#  - quotewrap only strings with quotes ('") or spaces from now on
#  - add --verbosity only if set in profile conf
#  - bugfix #138: fix quoting when filtering params, thx Eric
#  - bugfix #137: relax version parsing regex
#
#  2.5.0 (25.09.2023)
#  - bugfix #136: "not compatible with duplicity 2.x", thx tengel, lds, Rhomeo
#    check for duplicity 2.1+ (2.0 broke implied commands), 
#    command line ui changed incompatibly
#  - filter in/excludes more strictly for more duplicity actions now
#  - replace '--file-to-restore' with '--path-to-restore'
#  - filter backup only params now
#
#  2.4.3 (05.05.2023)
#  - bugfix #134: workaround bash 4.2 and earlier read bug (thx Tavio Wong)
#
#  2.4.2 (19.01.2023)
#  - featreq #55: change to purgeAuto in systemd unit files (thx B.Foresman)
#  - featreq #56: systemd files should go in /etc, not /lib (thx B.Foresman)
#  - bugfix #133: read -N not available on macOS (thx Peter Torelli)
#
#  2.4.1 (09.09.2022)
#  - fixup duplicity links, moved to http://duplicity.us
#  - bugfix: duply hangs on awk version detection on OpenBSD (thx phthomas137)
#
#  2.4 (06.04.2022)
#  - bugfix #127: date_from_nsecs ignores format string
#  - bugfix #116: separators print date now too
#  - featreq #48: add purgeAuto command (see man page)
#  - replaced tab indents with 2spaces everywhere
#  - bugfix #129,131,132: duply stumbles over 'python -s' shebang,
#      python interpreter parse failed if duplicity is a snap app
#  - bugfix #130: duplicity version check failed "gpg: WARNING: ..."
#  - version output, always print PYTHONPATH, if interpreter was determined
#  - update python references to python3
#
#  2.3.1 (11.02.2021)
#  - bugfix 123: symmetric encryption errs out, asks for '' private key
#
#  2.3 (30.12.2020)
#  - don't import whole key pair anymore if only pub/sec is requested
#  - gpg import routine informs on missing key files in profile now
#  - add check/import needed secret key for decryption
#  - featreq 50: Disable GPG key backups, implemented/added settings
#      GPG_IMPORT/GPG_EXPORT='disabled' to conf template
#
#  2.2.2 (24.02.2020)
#  - bugfix 120: Failures in "Autoset trust of key" during restore 
#    because of gpg2.2 fingerprint output change
#
#  2.2.1 (22.01.2020)
#  - featreq 46: Example systemd units & Howto, courtesy of Jozef Riha
#  - featreq 47: Clarify message about keeping the profile, also by Jozef Riha
#  - fix abbreviation spelling of 'e.g.'
#
#  2.2 (30.12.2018)
#  - featreq 44: implement grouping for batch commands
#      new separators are [] (square brackets) or groupIn/groupOut
#      command 'backup' translates now to [pre_bkp_post] to be skipped as
#      one block in case a condition was set in the batch instruction
#
#  2.1 (23.07.2018)
#  - be more verbose when duplicity version detection fails
#  - using info shows python binary's path for easier identification now
#  - reworked python interpreter handling, it's either
#      configured per PYTHON var
#      unconfigured, parsed from duplicity shebang
#      or set to current duplicity default 'python2' (was 'python' until now)
#  - do not quotewrap strings because of slashes (e.g. paths) anymore
#  - bugfix: improved in/exclude stripping from conf DUPL_PARAMS
#
#  2.0.4 (20.02.2018)
#  - bugfix 114: "duply usage is not current" wrt. purgeFull/Incr
#  - bugfix 115: typo in error message - "Not GPG_KEY entries" should be "No"
#  - bugfix 117: no duply_ prefix when ARCH_DIR is set in conf
#  - bugfix debian 882159: duply: occasionally shows negative runtimes
#
#  2.0.3 (29.08.2017)
#  - bugfix: "line 2231: CMDS: bad array subscript"
#  - bugfix 112: "env: illegal option -- u" on MacOSX
#
#  2.0.2 (23.05.2017)
#  - bugfix: never insert creds into file:// targets
#  - bugfix: avail profiles hint sometimes shortend the names by one char
#  - bugfix 108: CMD_NEXT variable should ignore conditional commands (and, or)
#  - export condition before/after next/prev command as CND_PREV,CND_NEXT now
#  - bugfix 97: Unknown command should be ERROR, not WARNING 
#
#  2.0.1 (16.11.2016)
#  - bugfix 104: Duply 2.0 sets wrong archive dir, --name was always 'duply_'
#
#  2.0 (27.10.2016)
#  made this a major version change, as we broke backward compatibility anyway
#  (see last change in v1.10). got complaints that rightfully pointed out
#  that should only come w/ a major version change. so, here we go ;)
#  if your backend stops working w/ this version create a new profile and
#  export the env vars needed as described in the comments of the conf file
#  directly above the SOURCE setting.
#  Changes:
#  - making sure multi spaces in TARGET survive awk processing
#  - new env var PROFILE exported to scripts 
#  - fix 102: expose a unique timestamp variable for pre/post scripts
#    actually a featreq. exporting RUN_START nanosec unix timestamp
#  - fix 101: GPG_AGENT_INFO is 'bogus' (thx Thomas Harning Jr.)
#  - fix 96: duply cannot handle two consecutive spaces in paths
#
#  1.11.3 (29.5.2016)
#  - fix wrong "WARNING: No running gpg-agent ..." when sign key was not set
#
#  1.11.2 (11.2.2016)
#  - fix "gpg: unsafe" version print out 
#  - bugfix 91: v1.11 [r47] broke asymmetric encryption when using GPG_KEYS_ENC
#  - bugfix 90: S3: TARGET_USER/PASS have no effect, added additional 
#    documentation about needed env vars to template conf file
#
#  1.11.1 (18.12.2015)
#  - bugfix 89: "Duply has trouble with PYTHON-interpreter" on OSX homebrew
#  - reverted duply's default PYTHON to 'python'
#
#  1.11 (24.11.2015)
#  - remove obsolete --ssh-askpass routine
#  - add PYTHON conf var to allow global override of used python interpreter
#  - enforced usage of "python2" in PATH as default interpreter for internal
#    use _and_ to run duplicity (setup.py changed the shebang to the fixed
#    path /usr/bin/python until 0.7.05, which we circumvent this way)
#  - featreq 36: support gpg-connect-agent as a means to detect if an agent is 
#    running (thx Thomas Harning Jr.), used gpg-agent for detection though
#  - quotewrapped run_cmd parameters to protect it from spaces e.g. in TMP path
#  - key export routine respects gpg-agent usage now
#
#  1.10.1 (19.8.2015)
#  - bugfix 86: Duply+Swift outputs warning
#  - bugfix 87: Swift fails without BACKEND_URL
#
#  1.10 (31.7.2015)
#  - featreq 37: busybox issues - fix awk, grep version detection,
#    fix grep failure because --color=never switch is unsupported
#    (thx Thomas Harning Jr. for reporting and helping to debug/fix it)
#  - bugfix 81: --exclude-globbing-filelist is deprecated since 0.7.03
#    (thx Joachim Wiedorn, also for maintaining the debian package)
#  - implemented base-/dirname as bash functions
#  - featreq 31 " Support for duplicity Azure backend " - ignored a 
#    contributed patch by Scott McKenzie and instead opted for removing almost
#    all code that deals with special env vars required by backends.
#    adding and modifying these results in too much overhead so i dropped this
#    feature. the future alternative for users is to consult the duplicity 
#    manpage and add the needed export definitions to the conf file.
#    appended a commented example to the template conf below the auth section.
#
#  1.9.2 (21.6.2015)
#  - bugfix: exporting keys with gpg2.1 works now (thx Philip Jocks)
#  - documented GPG_OPTS needed for gpg2.1 to conf template (thx Troy Engel)
#  - bugfix 82: GREP_OPTIONS=--color=always disrupted time calculation
#  - added GPG conf var (see conf template for details)
#  - added grep version output as it is an integral needed binary
#  - added PYTHONPATH printout in version output
#
#  1.9.1 (13.10.2014)
#  - export CMD_ERR now for scripts to detect if CMD_PREV failed/succeeded
#  - bugfix: CMD_PREV contained command even if it was skipped
#
#  1.9.0 (24.8.2014)
#  - bugfix: env vars were not exported when external script was executable
#  - rework GPG_KEY handling, allow virtually anything now (uid, keyid etc.) 
#    see gpg manpage, section "How to specify a user ID"
#    let gpg complain when the delivered values are invalid for whatever reason
#  - started to rework tmp space checking, exposed folder & writable check
#    TODO: reimplement enough file space available checking
#
#  1.8.0 (13.7.2014)
#  - add command verifyPath to expose 'verify --file-to-restore' action
#  - add time parameter support to verify command
#  - add section time formats to usage output 
#
#  1.7.4 (24.6.2014)
#  - remove ubuntu one support, service is discontinued
#  - featreq 31: add authenticated swift (contributed by Justus Seifert)
#
#  1.7.3 (3.4.2014)
#  - bugfix: test routines, gpg2 asked for passphrase although GPG_PW was set
#
#  1.7.2 (1.4.2014 "April,April")
#  - bugfix: debian Bug#743190 "duply no longer allows restoration without 
#     gpg passphrase in conf file"
#     GPG_AGENT_INFO env var is now needed to trigger --use-agent
#  - bugfix: gpg keyenc test routines didn't work if GPG_PW was not set
#
#  1.7.1 (30.3.2014)
#  - bugfix: purge-* commands renamed to purgeFull, purgeIncr due to 
#     incompatibility with new minus batch separator 
#
#  1.7.0 (20.3.2014)
#  - disabled gpg key id plausibility check, too many valid possibilities
#  - featreq 7 "Halt if precondition fails":
#     added and(+), or(-) batch command(separator) support
#  - featreq 26 "pre/post script with shebang line": 
#     if a script is flagged executable it's executed in a subshell 
#     now as opposed to sourced to bash, which is the default
#  - bugfix: do not check if dpbx, swift credentials are set anymore 
#  - bugfix: properly escape profile name, archdir if used as arguments
#  - add DUPL_PRECMD conf setting for use with e.g. trickle
#
#  1.6.0 (1.1.2014)
#  - support gs backend
#  - support dropbox backend
#  - add gpg-agent support to gpg test routines
#  - autoenable --use-agent if passwords were not defined in config
#  - GPG_OPTS are now honored everywhere, keyrings or complete gpg
#    homedir can thus be configured to be located anywhere
#  - always import both secret and public key if avail from config profile
#  - new explanatory comments in initial exclude file
#  - bugfix 7: Duply only imports one key at a time 
#
#  1.5.11 (19.07.2013)
#  - purge-incr command for remove-all-inc-of-but-n-full feature added
#    patch provided by Moritz Augsburger, thanks!
#  - documented version command in man page
#
#  1.5.10 (26.03.2013)
#  - minor indent and documentation fixes
#  - bugfix: exclude filter failed on ubuntu, mawk w/o posix char class support
#  - bugfix: fix url_decoding generally and for python3
#  - bugfix 3609075: wrong script results in status line (thx David Epping)
#
#  1.5.9 (22.11.2012)
#  - bugfix 3588926: filter --exclude* params for restore/fetch ate too much
#  - restore/fetch now also ignores --include* or --exclude='foobar' 
#
#  1.5.8 (26.10.2012)
#  - bugfix 3575487: implement proper cloud files support
#
#  1.5.7 (10.06.2012)
#  - bugfix 3531450: Cannot use space in target URL (file:///) anymore
#
#  1.5.6 (24.5.2012)
#  - commands purge, purge-full have no default value anymore for security 
#    reasons; instead max value can be given via cmd line or must be set
#    in profile; else an error is shown.
#  - minor man page modifications
#
#  versioning scheme will be simplified to [major].[minor].[patch] version
#  with the next version raise
#
#  1.5.5.5 (4.2.2012)
#  - bugfix 3479605: SEL context confused profile folder's permission check
#  - colon ':' in url passphrase got ignored, added python driven url_decoding
#    for user & pass to better process special chars
#
#  1.5.5.4 (16.10.2011)
#  - bugfix 3421268: SFTP passwords from conf ignored and always prompted for
#  - add support for separate sign passphrase (needs duplicity 0.6.14+)
#
#  1.5.5.3 (1.10.2011)
#  - bugfix 3416690: preview threw echo1 error
#  - fix unknown cmds error usage & friends if more than 2 params were given
#
#  1.5.5.2 (23.9.2011)
#  - bugfix 3409643: ssh key auth did ask for passphrase (--ssh-askpass ?)
#  - bugfix: mawk does not support \W and did not split multikey definitions
#  - all parameters should survive  single (') and double (") quotes now
#
#  1.5.5.1 (7.6.2011)
#  - featreq 3311881: add ftps as supported by duplicity 0.6.13 (thx mape2k)
#  - bugfix 3312208: signing detection broke symmetric gpg test routine
#
#  1.5.5 (2.5.2011)
#  - bugfix: fetch problem with space char in path, escape all params 
#    containing non word chars
#  - list available profiles, if given profile cannot be found
#  - added --use-agent configuration hint
#  - bugfix 3174133: --exclude* params in conf DUPL_PARAMS broke 
#    fetch/restore
#  - version command now prints out 'using installed' info
#  - featreq 3166169: autotrust imported keys, based on code submitted by 
#    Martin Ellis - imported keys are now automagically trusted ultimately 
#  - new txt2man feature to create manpages for package maintainers
#
#  1.5.4.2 (6.1.2011)
#  - new command changelog
#  - bugfix 3109884: freebsd awk segfaulted on printf '%*', use print again
#  - bugfix: freebsd awk hangs on 'awk -W version' 
#  - bugfix 3150244: mawk does not know '--version'
#  - minor help text improvements
#  - new env vars CMD_PREV,CMD_NEXT replacing CMD env var for scripts
#
#  1.5.4.1 (4.12.2010)
#  - output awk, python, bash version now in prolog
#  - shebang uses /usr/bin/env now for freebsd compatibility, 
#    bash not in /bin/bash 
#  - new --disable-encryption parameter, 
#    to override profile encr settings for one run
#  - added exclude-if-present setting to conf template
#  - bug 3126972: GPG_PW only needed for signing/symmetric encryption 
#    (even though duplicity still needs it)
#
#  1.5.4 (15.11.2010)
#  - as of 1.5.3 already, new ARCH_DIR config option
#  - multiple key support
#  - ftplicity-Feature Requests-2994929: separate encryption and signing key
#  - key signing of symmetric encryption possible (duplicity patch committed)
#  - gpg tests disable switch
#  - gpg tests now previewable and more intelligent
#
#  1.5.3 (1.11.2010)
#  - bugfix 3056628: improve busybox compatibility, grep did not have -m param
#  - bugfix 2995408: allow empty password for PGP key
#  - bugfix 2996459: Duply erroneously escapes '-' symbol in username
#  - url_encode function is now pythonized
#  - rsync uses FTP_PASSWORD now if duplicity 0.6.10+ , else issue warning
#  - feature 3059262: Make pre and post aware of parameters, 
#                     internal parameters + CMD of pre or post 
#
#  1.5.2.3 (16.4.2010)
#  - bugfix: date again, should now work virtually anywhere
#
#  1.5.2.2 (3.4.2010)
#  - minor bugfix: duplicity 0.6.8b version string now parsable
#  - added INSTALL.txt
#
#  1.5.2.1 (23.3.2010)
#  - bugfix: date formatting is awked now and should work on all platforms
#
#  1.5.2 (2.3.2010)
#  - bugfix: errors print to STD_ERR now, failed tasks print an error message
#  - added --name=duply_<profile> for duplicity 0.6.01+ to name cache folder
#  - simplified & cleaned profileless commands, removed second instance
#  - generalized separator time routines
#  - added support for --no-encryption (GPG_KEY='disabled'), see conf examples
#  - minor fixes
#
#  1.5.1.5 (5.2.2010)
#  - bugfix: added special handling of credentials for rsync, imap(s)
#
#  1.5.1.4 (7.1.2010)
#  - bugfix: nsecs defaults now to zeroes if date does not deliver [0-9]{9}
#  - check if ncftp binary is available if url protocol is ftp
#  - bugfix: duplicity output is now printed to screen directly to resolve
#            'mem alloc problem' bug report
#  - bugfix: passwords will not be in the url anymore to solve the 'duply shows
#            sensitive data in process listing' bug report
#
#  1.5.1.3 (24.12.2009) 'merry xmas'
#  - bugfix: gpg pass now apostrophed to allow space and friends
#  - bugfix: credentials are now url encoded to allow special chars in them
#            a note about url encoding has been added to the conf template
#
#  1.5.1.2 (1.11.2009)
#  - bugfix: open parenthesis in password broke duplicity execution
#  - bugfix: ssh/scp backend does not always need credentials e.g. key auth
#
#  1.5.1.1 (21.09.2009)
#  - bugfix: fixed s3[+http] TARGET_PASS not needed routine
#  - bugfix: TYPO in duply 1.5.1 prohibited the use of /etc/duply
#    see https://sourceforge.net/tracker/index.php?func=detail&
#                aid=2864410&group_id=217745&atid=1041147
#
#  1.5.1 (21.09.2009) - duply (fka. ftplicity)
#  - first things first: ftplicity (being able to support all backends since 
#    some time) will be called duply (fka. ftplicity) from now on. The addendum
#    is for the time being to circumvent confusion.
#  - bugfix: exit code is 1 (error) not 0 (success), if at least on duplicity 
#            command failed
#  - s3[+http] now supported natively by translating user/pass to access_key/
#    secret_key environment variables needed by duplicity s3 boto backend 
#  - bugfix: additional output lines do not confuse version check anymore
#  - list command supports now age parameter (patch by stefan on feature 
#    request tracker)
#  - bugfix: option/param pairs are now correctly passed on to duplicity
#  - bugfix: s3[+http] needs no TARGET_PASS if command is read only
#
#  1.5.0.2 (31.07.1009)
#  - bugfix: insert password in target url didn't work with debian mawk
#            related to previous bug report
#
#  1.5.0.1 (23.07.2009)
#  - bugfix: gawk gensub dependency raised an error on debian's default mawk
#            replaced with match/substr command combination (bug report)
#            https://sf.net/tracker/?func=detail&atid=1041147&aid=2825388&
#            group_id=217745
#
#  1.5.0 (01.07.2009)
#  - removed ftp limitation, all duplicity backends should work now
#  - bugfix: date for separator failed on openwrt busybox date, added a 
#    detecting workaround, milliseconds are not available w/ busybox date
#
#  1.4.2.1 (14.05.2009)
#  - bugfix: free temp space detection failed with lvm, fixed awk parse routine
#
#  1.4.2 (22.04.2009)
#  - gpg keys are now exported as gpgkey.[id].asc , the suffix reflects the
#    armored ascii nature, the id helps if the key is switched for some reason
#    im/export routines are updated accordingly (import is backward compatible 
#    to the old profile/gpgkey files)         
#  - profile argument is treated as path if it contains slashes 
#    (for details see usage)
#  - non-ftplicity options (all but --preview currently) are now passed 
#    on to duplicity 
#  - removed need for stat in secure_conf, it is ls based now
#  - added profile folder readable check
#  - added gpg version & home info output
#  - awk utility availability is now checked, because it was mandatory already
#  - tmp space is now checked on writability and space requirement
#    test fails on less than 25MB or configured $VOLSIZE, 
#    test warns if there is less than two times $VOLSIZE because 
#    that's required for --asynchronous-upload option  
#  - gpg functionality is tested now before executing duplicity 
#    test drive contains encryption, decryption, comparison, cleanup
#    this is meant to detect non trusted or other gpg errors early
#  - added possibility of doing symmetric encryption with duplicity
#    set GPG_KEY="" or simply comment it out
#  - added hints in config template on the depreciation of 
#    --short-filenames, --time-separator duplicity options
#
#  new versioning scheme 1.4.2b => 1.4.2, 
#  beta b's are replaced by a patch count number e.g. 1.4.2.1 will be assigned
#  to the first bug fixing version and 1.4.2.2 to the second and so on
#  also the releases will now have a release date formatted (Day.Month.Year)
#
#  1.4.1b1 - bugfix: ftplicity changed filesystem permission of a folder
#            named exactly as the profile if existing in executing dir
#          - improved plausibility checking of config and profile folder
#          - secure_conf only acts if needed and prints a warning now
#
#  1.4.1b  - introduce status (duplicity collection-status) command
#          - pre/post script output printed always now, not only on errors
#          - new config parameter GPG_OPTS to pass gpg options
#            added examples & comments to profile template conf
#          - reworked separator times, added duration display
#          - added --preview switch, to preview generated command lines
#          - disabled MAX_AGE, MAX_FULL_BACKUPS, VERBOSITY in generated
#            profiles because they have reasonable defaults now if not set
#
#  1.4.0b1 - bugfix: incr forces incremental backups on duplicity,
#            therefore backup translates to pre_bkp_post now
#          - bugfix: new command bkp, which represents duplicity's 
#            default action (incr or full if full_if_older matches
#            or no earlier backup chain is found)
#
#  new versioning scheme 1.4 => 1.4.0, added new minor revision number
#  this is meant to slow down the rapid version growing but still keep 
#  versions cleanly separated.
#  only additional features will raise the new minor revision number. 
#  all releases start as beta, each bugfix release will raise the beta 
#  count, usually new features arrive before a version 'ripes' to stable
#    
#  1.4.0b
#    1.4b  - added startup info on version, time, selected profile
#          - added time output to separation lines
#          - introduced: command purge-full implements duplicity's 
#            remove-all-but-n-full functionality (patch by unknown),
#            uses config variable $MAX_FULL_BACKUPS (default = 1)
#          - purge config var $MAX_AGE defaults to 1M (month) now 
#          - command full does not execute pre/post anymore
#            use batch command pre_full_post if needed 
#          - introduced batch mode cmd1_cmd2_etc
#            (in turn removed the bvp command)
#          - unknown/undefined command issues a warning/error now
#          - bugfix: version check works with 0.4.2 and older now
#    1.3b3 - introduced pre/post commands to execute/debug scripts
#          - introduced bvp (backup, verify, purge)
#          - bugfix: removed need for awk gensub, now mawk compatible
#    1.3b2 - removed pre/post need executable bit set 
#          - profiles now under ~/.ftplicity as folders
#          - root can keep profiles in /etc/ftplicity, folder must be
#            created by hand, existing profiles must be moved there
#          - removed ftplicity in path requirement
#          - bugfix: bash < v.3 did not know '=~'
#          - bugfix: purge works again 
#    1.3   - introduces multiple profiles support
#          - modified some script errors/docs
#          - reordered gpg key check import routine
#          - added 'gpg key id not set' check
#          - added error_gpg (adds how to setup gpg key howto)
#          - bugfix: duplicity 0.4.4RC4+ parameter syntax changed
#          - duplicity_version_check routine introduced
#          - added time separator, shortnames, volsize, full_if_older 
#            duplicity options to config file (inspired by stevie 
#            from http://weareroot.de) 
#    1.1.1 - bugfix: encryption reactivated
#    1.1   - introduced config directory
#    1.0   - first release
################################################################################

# utility functions overriding binaries

# wrap grep to override possible env set GREP_OPTIONS=--color=always
function grep {
  command env "GREP_OPTIONS=" grep "$@"
}

# implement basename in plain bash
function basename {
  local stripped="${1%/}"
  echo "${stripped##*/}"
}

# implement dirname in plain bash
function dirname {
  echo ${1%/*}
}

# implement basic which in plain bash
function which {
  type -p "$@"
}

# check availability of executables via file name or file paths
function lookup {
  local bin="$1"
  # look for file names in path via bash hash OR 
  # look for executables at given relative/absolute location
  ( [ "${bin##*/}" == "$bin" ] && hash "$bin" 2>/dev/null ) || [ -x "$bin" ]
}

# important definitions #######################################################

ME_LONG="$0"
ME="$(basename $0)"
ME_NAME="${ME%%.*}"
ME_VERSION="2.5.2"
ME_WEBSITE="https://duply.net"

# default config values
DEFAULT_SOURCE='/path/of/source'
DEFAULT_TARGET='scheme://user[:password]@host[:port]/[/]path'
DEFAULT_TARGET_USER='_backend_username_'
DEFAULT_TARGET_PASS='_backend_password_'
DEFAULT_GPG='gpg'
DEFAULT_GPG_KEY='_KEY_ID_'
DEFAULT_GPG_PW='_GPG_PASSWORD_'

# function definitions ##########################

function set_config { # sets global config vars
  local CONFHOME_COMPAT="$HOME/.ftplicity"
  local CONFHOME="$HOME/.duply"
  local CONFHOME_ETC_COMPAT="/etc/ftplicity"
  local CONFHOME_ETC="/etc/duply"

  # confdir can be delivered as path (must contain /)
  if [ `echo $FTPLCFG | grep /` ] ; then 
    CONFDIR=$(readlink -f $FTPLCFG 2>/dev/null || \
              ( echo $FTPLCFG|grep -v '^/' 1>/dev/null 2>&1 \
               && echo $(pwd)/${FTPLCFG} ) || \
              echo ${FTPLCFG})          
  # or DEFAULT in home/.duply folder (NEW)
  elif [ -d "${CONFHOME}" ]; then
    CONFDIR="${CONFHOME}/${FTPLCFG}"
  # or in home/.ftplicity folder (OLD)
  elif [ -d "${CONFHOME_COMPAT}" ]; then
    CONFDIR="${CONFHOME_COMPAT}/${FTPLCFG}"
    warning_oldhome "${CONFHOME_COMPAT}" "${CONFHOME}"
  # root can put profiles under /etc/duply (NEW) if path exists
  elif [ -d "${CONFHOME_ETC}" ] && [ "$EUID" -eq 0 ]; then
    CONFDIR="${CONFHOME_ETC}/${FTPLCFG}"
  # root can keep profiles under /etc/ftplicity (OLD) if path exists
  elif [ -d "${CONFHOME_ETC_COMPAT}" ] && [ "$EUID" -eq 0 ]; then
    CONFDIR="${CONFHOME_ETC_COMPAT}/${FTPLCFG}"
    warning_oldhome "${CONFHOME_ETC_COMPAT}" "${CONFHOME_ETC}"
  # hmm no profile folder there, then use default for error later
  else
    CONFDIR="${CONFHOME}/${FTPLCFG}" # continue, will fail later in main
  fi

  # remove trailing slash, get profile name etc.
  CONFDIR="${CONFDIR%/}"
  PROFILE="${CONFDIR##*/}"
  CONF="$CONFDIR/conf"
  PRE="$CONFDIR/pre"
  POST="$CONFDIR/post"
  EXCLUDE="$CONFDIR/exclude"
  KEYFILE="$CONFDIR/gpgkey.asc"
}

function version_info { # print version information
  cat <<END
  $ME_NAME version $ME_VERSION
  ($ME_WEBSITE)
END
}

function version_info_using { 
  cat <<END
$(version_info)

  $(using_info)
END
}

function using_info {
  # init needed vars into global name space
  lookup duplicity && { duplicity_version_get; }
  local NOTFOUND="INVALID"
  local AWK_VERSION GREP_VERSION PYTHON_RUNNER \
    PYTHON_RUNNER_RESOLVED PYTHON_VERSION PYTHON_PATH
  # freebsd awk / GNU awk (--version), 
  # debian mawk (-W version), 
  # openbsd awk (-V, exitcode 0 when any program string is given regardless .e.g. "-W version", so place it last) 
  # some awks wait for input if they misinterpret/don't know the options, pipe '' as a precaution
  AWK_VERSION=$( lookup awk && ( 
	echo | awk --version ||\
	echo | awk -V ||\
	echo | awk -W version ) 2>/dev/null | awk 'NR<=2&&tolower($0)~/(busybox|awk)/{success=1;print;exit} END{if(success<1) print "unknown"}' || echo "$NOTFOUND" )
  GREP_VERSION=$( lookup grep && grep --version 2>&1 | awk 'NR<=2&&tolower($0)~/(busybox|grep.*[0-9]+\.[0-9]+)/{success=1;print;exit} END{if(success<1) print "unknown"}' || echo "$NOTFOUND" )

  if [ -n "$PYTHON" ]; then
    PYTHON_RUNNER=$PYTHON
  else
    PYTHON_RUNNER="$(duplicity_python_binary_parse)"
  fi
  # fetch version and resolve python
  [ -n "$PYTHON_RUNNER" ] && {
    PYTHON_VERSION=$($PYTHON_RUNNER -V 2>&1| awk '{print tolower($0);exit}' || echo "'$PYTHON_RUNNER' $NOTFOUND" )
    local PYTHON_RUNNER_ARRAY=( $PYTHON_RUNNER )
    PYTHON_RUNNER_RESOLVED="$(which ${PYTHON_RUNNER_ARRAY[0]})"
    # readd params if there were
    [ ${#PYTHON_RUNNER_ARRAY[@]} -gt 1 ] && \
      PYTHON_RUNNER_RESOLVED="${PYTHON_RUNNER_RESOLVED} ${PYTHON_RUNNER_ARRAY[@]:1}"
      PYTHON_PATH="$($PYTHON_RUNNER -c "import sys;print(':'.join(sys.path));")"
  }

  local GPG_INFO=$(gpg_avail && gpg --version 2>&1| awk '/^gpg.*[0-9\.]+$/&&length(v)<1{v=$1" "$3}/^Home:/{h=" ("$0")"}END{print v""h}' || echo "gpg $NOTFOUND")
  local BASH_VERSION=$(bash --version | awk 'NR==1{IGNORECASE=1;sub(/GNU bash, version[ ]+/,"",$0);print $0}')
  # print out
  echo -e "Using installed duplicity version ${DUPL_VERSION:-$NOTFOUND}\
${PYTHON_VERSION+, $PYTHON_VERSION ${PYTHON_RUNNER:+($PYTHON_RUNNER_RESOLVED)}${PYTHON_PATH:+ 'PYTHONPATH=$PYTHON_PATH'}}\
${GPG_INFO:+, $GPG_INFO}${AWK_VERSION:+, awk '${AWK_VERSION}'}${GREP_VERSION:+, grep '${GREP_VERSION}'}\
${BASH_VERSION:+, bash '${BASH_VERSION}'}."
}

function usage_info { # print usage information

  cat <<USAGE_EOF
VERSION:
$(version_info)
  
DESCRIPTION: 
  Duply deals as a wrapper for the mighty duplicity magic.
  It simplifies running duplicity with cron or on command line by:

    - keeping recurring settings in profiles per backup job
    - enabling batch operations e.g. backup_verify+purge
    - executing pre/post scripts (different actions possible 
      depending on previous or next command or it's exit status)
    - precondition checking for flawless duplicity operation

  For each backup job one configuration profile must be created.
  The profile folder will be stored under '~/.${ME_NAME}/<profile>'
  (where ~ is the current users home directory).
  Hint:  
   If the folder '/etc/${ME_NAME}' exists, the profiles for the super
   user root will be searched & created there.

USAGE:
  first time usage (profile creation):  
    $ME <profile> create

  general usage in single or batch mode (see EXAMPLES):  
    $ME <profile> <command>[[_|+|-]<command>[_|+|-]...] [<options> ...]

  For batches the conditional separators can also be written as pseudo commands
  and(+), or(-). See SEPARATORS for details.

  Non $ME options are passed on to duplicity (see OPTIONS).
  All conf parameters can also be defined in the environment instead.

PROFILE:
  Indicated by a path or a profile name (<profile>), which is resolved 
  to '~/.${ME_NAME}/<profile>' (~ expands to environment variable \$HOME).

  Superuser root can place profiles under '/etc/${ME_NAME}'. Simply create
  the folder manually before running $ME_NAME as superuser.
  Note:  
    Already existing profiles in root's home folder will cease to work
    unless they are moved to the new location manually.

  example 1:   $ME humbug backup

  Alternatively a _path_ might be used e.g. useful for quick testing, 
  restoring or exotic locations. Shell expansion should work as usual.
  Hint:  
    The path must contain at least one path separator '/', 
    e.g. './test' instead of only 'test'.

  example 2:   $ME ~/.${ME_NAME}/humbug backup

SEPARATORS:
  _ (underscore)  
             neutral separator
  + (plus sign), _and_  
             conditional AND
             the next command will only be executed if the previous succeeded
  - (minus sign), _or_  
             conditional OR
             the next command will only be executed if the previous failed
  [] (square brackets), _groupIn_/_groupOut_  
             enables grouping of commands

   example:  
    'pre+[bkp-verify]_post' translates to
    'pre_and_groupIn_bkp_or_verify_groupOut_post'

COMMANDS:
  usage      get usage help text

  and/or/groupIn/groupOut  
             pseudo commands used in batches (see SEPARATORS above)

  create     creates a configuration profile
  backup     backup with pre/post script execution (batch: [pre_bkp_post]),
              full (if full_if_older matches or no earlier backup is found)
              incremental (in all other cases)
  pre/post   execute '<profile>/$(basename "$PRE")', '<profile>/$(basename "$POST")' scripts
  bkp        as above but without executing pre/post scripts
  full       force full backup
  incr       force incremental backup
  list [<age>]  
             list all files in backup (as it was at <age>, default: now)
  status     prints backup sets and chains currently in repository
  verify [<age>] [--compare-data]  
             list files changed, since age if given
  verifyPath <rel_path_in_bkp> <local_path> [<age>] [--compare-data]  
             list changes of a file or folder path in backup compared to a
             local path, since age if given
  restore <target_path> [<age>]  
             restore the complete backup to <target_path> [as it was at <age>]
  fetch <src_path> <target_path> [<age>]  
             fetch single file/folder from backup [as it was at <age>]
  purge [<max_age>] [--force]  
             list outdated backup files (older than \$MAX_AGE)
              [use --force to actually delete these files]
  purgeFull [<max_full_backups>] [--force]  
             list outdated backup files (\$MAX_FULL_BACKUPS being the number of
             full backups and associated incrementals to keep, counting in 
             reverse chronological order)
              [use --force to actually delete these files]
  purgeIncr [<max_fulls_with_incrs>] [--force]  
             list outdated incremental backups (\$MAX_FULLS_WITH_INCRS being 
             the number of full backups which associated incrementals will be
             kept, counting in reverse chronological order) 
              [use --force to actually delete these files]
  purgeAuto [--force]  
             convenience batch wrapper for all purge commands above.
             purge, purgeFull, purgeIncr are added if their conf vars were set. e.g.
              MAX_AGE=1Y
              MAX_FULL_BACKUPS=6
              MAX_FULLS_WITH_INCR=3
             in profile conf file would result in
              [purge_purgeFull_purgeIncr]
  cleanup [--force]  
             list broken backup chain files archives (e.g. after unfinished run)
              [use --force to actually delete these files]

  changelog  print changelog / todo list
  txt2man    feature for package maintainers - create a manpage based on the 
             usage output. download txt2man from http://mvertes.free.fr/, put 
             it in the PATH and run '$ME txt2man' to create a man page.
  version    show version information of $ME_NAME and needed programs

OPTIONS:
  --force    passed to duplicity (see commands: 
             purge, purgeFull, purgeIncr, cleanup)
  --preview  do nothing but print out generated duplicity command lines
  --disable-encryption  
             disable encryption, overrides profile settings

TIME FORMATS:
  For all time related parameters like age, max_age etc.
  Refer to the duplicity manpage for all available formats. Here some examples:
    2002-01-25T07:00:00+02:00 (full date time format string)
    2002/3/5 (date string YYYY/MM/DD)
    12D (interval, 12 days ago)
    1h78m (interval, 1 hour 78 minutes ago)

PRE/POST SCRIPTS:
  Some useful internal duply variables are exported to the scripts.

    PROFILE, CONFDIR, SOURCE, TARGET_URL_<PROT|HOSTPATH|USER|PASS>, 
    GPG_<KEYS_ENC|KEY_SIGN|PW>, CMD_ERR, RUN_START,
    CMD_<PREV|NEXT> (previous/next command), 
    CND_<PREV|NEXT> (condition before/after)

  The CMD_* variables were introduced to allow different actions according to 
  the command the scripts were attached to e.g. 'pre_bkp_post_pre_verify_post' 
  will call the pre script two times, with CMD_NEXT variable set to 'bkp' 
  on the first and to 'verify' on the second run.
  CMD_ERR holds the exit code of the CMD_PREV .

EXAMPLES:
  create profile 'humbug':  
    $ME humbug create (don't forget to edit this new conf file)
  backup 'humbug' now:  
    $ME humbug backup
  list available backup sets of profile 'humbug':  
    $ME humbug status
  list and delete outdated backups of 'humbug':  
    $ME humbug purge --force
  restore latest backup of 'humbug' to /mnt/restore:  
    $ME humbug restore /mnt/restore
  restore /etc/passwd of 'humbug' from 4 days ago to /root/pw:  
    $ME humbug fetch etc/passwd /root/pw 4D
    (see "duplicity manpage", section TIME FORMATS)
  a one line batch job on 'humbug' for cron execution:  
    $ME humbug backup_verify_purge --force
  batch job to run a full backup with pre/post scripts:  
    $ME humbug pre_full_post

FILES:
  in profile folder '~/.${ME_NAME}/<profile>' or '/etc/${ME_NAME}'
  conf             profile configuration file
  pre,post         pre/post scripts (see above for details)
  gpgkey.*.asc     exported GPG key files
  exclude          a globbing list of included or excluded files/folders
                   (see "duplicity manpage", section FILE SELECTION)

$(hint_profile)

SEE ALSO:
  duplicity man page duplicity(1) or http://duplicity.us/docs.html
USAGE_EOF
}

# to check call 'duply txt2man | man -l -'
function usage_txt2man {
  usage_info | \
  awk '/^^[^[:lower:][:space:]][^[:lower:]]+$/{gsub(/[^[:upper:]]/," ",$0)}{print}' |\
  txt2man -t"$(toupper "${ME_NAME}")" -s1 -r"${ME_NAME}-${ME_VERSION}" -v'User Manuals'
}

function changelog {
  cat $ME_LONG | awk '/^#####/{on=on+1}(on==3){sub(/^#(  )?/,"",$0);print}'
}

function create_config {
  if [ ! -d "$CONFDIR" ] ; then
    mkdir -p "$CONFDIR" || error "Couldn't create config '$CONFDIR'."
  # create initial config file
    cat <<EOF >"$CONF"
# gpg encryption settings, simple settings:
#  GPG_KEY='disabled' - disables encryption alltogether
#  GPG_KEY='<key1>[,<key2>]'; GPG_PW='pass' - encrypt with keys,
#   sign if secret key of key1 is available use GPG_PW for sign & decrypt
#  Note: you can specify keys via all methods described in gpg manpage,
#        section "How to specify a user ID", escape commas (,) via backslash (\)
#        e.g. 'Mueller, Horst', 'Bernd' -> 'Mueller\, Horst, Bernd'
#        as they are used to separate the entries
#  GPG_PW='passphrase' - symmetric encryption using passphrase only
GPG_KEY='${DEFAULT_GPG_KEY}'
GPG_PW='${DEFAULT_GPG_PW}'
# gpg encryption settings in detail (extended settings)
#  the above settings translate to the following more specific settings
#  GPG_KEYS_ENC='<keyid1>[,<keyid2>,...]' - list of pubkeys to encrypt to
#  GPG_KEY_SIGN='<keyid1>|disabled' - a secret key for signing
#  GPG_PW='<passphrase>' - needed for signing, decryption and symmetric
#   encryption. If you want to deliver different passphrases for e.g. 
#   several keys or symmetric encryption plus key signing you can use
#   gpg-agent. Simply make sure that GPG_AGENT_INFO is set in environment.
#   also see "A NOTE ON SYMMETRIC ENCRYPTION AND SIGNING" in duplicity manpage 
# notes on en/decryption
#  private key and passphrase will only be needed for decryption or signing.
#  decryption happens on restore and incrementals (compare archdir contents).
#  for security reasons it makes sense to separate the signing key from the
#  encryption keys. https://answers.launchpad.net/duplicity/+question/107216
#GPG_KEYS_ENC='<pubkey1>,<pubkey2>,...'
#GPG_KEY_SIGN='<prvkey>'
# set if signing key passphrase differs from encryption (key) passphrase
# NOTE: available since duplicity 0.6.14, translates to SIGN_PASSPHRASE
#GPG_PW_SIGN='<signpass>'

# uncomment and set a file path or name force duply to use this gpg executable
# available in duplicity 0.7.04 and above (currently unreleased 06/2015)
#GPG='/usr/local/gpg-2.1/bin/gpg'

# gpg options passed from duplicity to gpg process (default='')
# e.g. "--trust-model pgp|classic|direct|always" 
#   or "--compress-algo=bzip2 --bzip2-compress-level=9"
#   or "--personal-cipher-preferences AES256,AES192,AES..."
#   or "--homedir ~/.duply" - keep keyring and gpg settings duply specific
#   or "--pinentry-mode loopback" - needed for GPG 2.1+ _and_
#      also enable allow-loopback-pinentry in your .gnupg/gpg-agent.conf
#GPG_OPTS=''

# disable preliminary tests with the following setting
#GPG_TEST='disabled'
# disable automatic gpg key importing altogether
#GPG_IMPORT='disabled'
# disable automatic gpg key exporting to profile folder
#GPG_EXPORT='disabled'

# backend, credentials & location of the backup target (URL-Format)
# generic syntax is
#   scheme://[user[:password]@]host[:port]/[/]path
# e.g.
#   sftp://bob:secret@backupserver.com//home/bob/dupbkp
# for details and available backends see duplicity manpage, section URL Format
#   http://duplicity.us/vers8/duplicity.1.html#url-format
# BE AWARE:
#   some backends (cloudfiles, S3 etc.) need additional env vars to be set to
#   work properly, read after the TARGET definition for more details.
# ATTENTION:
#   characters other than A-Za-z0-9.-_.~ in the URL have to be
#   replaced by their url encoded pendants, see
#     http://en.wikipedia.org/wiki/Url_encoding
#   if you define the credentials as TARGET_USER, TARGET_PASS below $ME
#   will try to url_encode them for you if the need arises.
TARGET='${DEFAULT_TARGET}'

# optionally the username/password can be defined as extra variables
# setting them here _and_ in TARGET results in an error
# ATTENTION:
#   there are backends that do not support the user/pass auth scheme.
#   prominent examples are S3, Azure, Cloudfiles. when in doubt consult the
#   duplicity manpage. usually there is a NOTE section explaining if and which
#   env vars should be set.
#TARGET_USER='${DEFAULT_TARGET_USER}'
#TARGET_PASS='${DEFAULT_TARGET_PASS}'
# e.g. for cloud files backend it might look like this (uncomment for use!)
#export CLOUDFILES_USERNAME='someuser'
#export CLOUDFILES_APIKEY='somekey'
#export CLOUDFILES_AUTHURL ='someurl'
# the following is an incomplete list (<backend>: comma separated env vars list)
# Azure: AZURE_ACCOUNT_NAME, AZURE_ACCOUNT_KEY
# Cloudfiles: CLOUDFILES_USERNAME, CLOUDFILES_APIKEY, CLOUDFILES_AUTHURL
# Google Cloud Storage: GS_ACCESS_KEY_ID, GS_SECRET_ACCESS_KEY
# Pydrive: GOOGLE_DRIVE_ACCOUNT_KEY, GOOGLE_DRIVE_SETTINGS
# S3: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
# Swift: SWIFT_USERNAME, SWIFT_PASSWORD, SWIFT_AUTHURL,
#        SWIFT_TENANTNAME OR SWIFT_PREAUTHURL, SWIFT_PREAUTHTOKEN

# base directory to backup
SOURCE='${DEFAULT_SOURCE}'

# a command that runs duplicity e.g. 
#  shape bandwidth use via trickle
#  "trickle -s -u 640 -d 5120" # 5Mb up, 40Mb down"
#DUPL_PRECMD=""

# override the python interpreter to execute duplicity, unset by default
#  e.g. "python3" or "/usr/bin/python3.8"
#PYTHON="python"

# exclude folders containing exclusion file (since duplicity 0.5.14)
# Uncomment the following two lines to enable this setting.
#FILENAME='.duplicity-ignore'
#DUPL_PARAMS="\$DUPL_PARAMS --exclude-if-present '\$FILENAME'"

# Time frame for old backups to keep, Used for the "purge" command.  
# see duplicity man page, chapter TIME_FORMATS)
#MAX_AGE=1M

# Number of full backups to keep. Used for the "purgeFull" command. 
# See duplicity man page, action "remove-all-but-n-full".
#MAX_FULL_BACKUPS=1

# Number of full backups for which incrementals will be kept for.
# Used for the "purgeIncr" command.
# See duplicity man page, action "remove-all-inc-of-but-n-full".
#MAX_FULLS_WITH_INCRS=1

# activates duplicity --full-if-older-than option (since duplicity v0.4.4.RC3) 
# forces a full backup if last full backup reaches a specified age, for the 
# format of MAX_FULLBKP_AGE see duplicity man page, chapter TIME_FORMATS
# Uncomment the following two lines to enable this setting.
#MAX_FULLBKP_AGE=1M
#DUPL_PARAMS="\$DUPL_PARAMS --full-if-older-than \$MAX_FULLBKP_AGE " 

# sets duplicity --volsize option (available since v0.4.3.RC7)
# set the size of backup chunks to VOLSIZE MB instead of the default 25MB.
# VOLSIZE must be number of MB's to set the volume size to.
# Uncomment the following two lines to enable this setting. 
#VOLSIZE=50
#DUPL_PARAMS="\$DUPL_PARAMS --volsize \$VOLSIZE "

# verbosity of output (error 0, warning 1-2, notice 3-4, info 5-8, debug 9)
# default is 4, if not set
#VERBOSITY=5

# temporary file space. at least the size of the biggest file in backup
# for a successful restoration process. (default is '/tmp', if not set)
#TEMP_DIR=/tmp

# Modifies archive-dir option (since 0.6.0) Defines a folder that holds 
# unencrypted meta data of the backup, enabling new incrementals without the 
# need to decrypt backend metadata first. If empty or deleted somehow, the 
# private key and it's password are needed.
# NOTE: This is confidential data. Put it somewhere safe. It can grow quite 
#       big over time so you might want to put it not in the home dir.
# default '~/.cache/duplicity/duply_<profile>/'
# if set  '\${ARCH_DIR}/<profile>'
#ARCH_DIR=/some/space/safe/.duply-cache

# DEPRECATED setting
# sets duplicity --time-separator option (since v0.4.4.RC2) to allow users 
# to change the time separator from ':' to another character that will work 
# on their system.  HINT: For Windows SMB shares, use --time-separator='_'.
# NOTE: '-' is not valid as it conflicts with date separator.
# ATTENTION: only use this with duplicity < 0.5.10, since then default file 
#            naming is compatible and this option is pending depreciation 
#DUPL_PARAMS="\$DUPL_PARAMS --time-separator _ "

# DEPRECATED setting
# activates duplicity --short-filenames option, when uploading to a file
# system that can't have filenames longer than 30 characters (e.g. Mac OS 8)
# or have problems with ':' as part of the filename (e.g. Microsoft Windows)
# ATTENTION: only use this with duplicity < 0.5.10, later versions default file 
#            naming is compatible and this option is pending depreciation
#DUPL_PARAMS="\$DUPL_PARAMS --short-filenames "

# more duplicity command line options can be added in the following way
# don't forget to leave a separating space char at the end
#DUPL_PARAMS="\$DUPL_PARAMS --put_your_options_here " 

EOF

# create initial exclude file
    cat <<EOF >"$EXCLUDE"
# although called exclude, this file is actually a globbing file list
# duplicity accepts some globbing patterns, even including ones here
# here is an example, this incl. only 'dir/bar' except it's subfolder 'foo'
# - dir/bar/foo
# + dir/bar
# - **
# for more details see duplicity manpage, section File Selection
# http://duplicity.us/vers8/duplicity.1.html#file-selection

EOF

  # Hints on first usage
  cat <<EOF

Congratulations. You just created the profile '$FTPLCFG'.
The initial config file has been created as 
'$CONF'.
You should now adjust this config file to your needs.

$(hint_profile)

EOF
fi

}

# used in usage AND create_config
function hint_profile {
  cat <<EOF
IMPORTANT:
  Copy the _whole_ profile folder after the first backup to a safe place.
  It contains everything (duply related) needed to restore your backups. 

  Pay attention to (possibly later added) external files such as credentials
  or auth files (e.g. netrc, .megarc, ssh keys) or environment variables
  (e.g. DPBX_ACCESS_TOKEN). 
  It is good policy to place those in the profile folder if possible at all.
    e.g. in case of 'multi://' target the config .json file
  Env vars should be added to duply profiles' conf file.

  Keep access to these files restricted as they contain information (gpg key,
  passphrases etc.) to access and modify your backups.
  
  Finally:
  You should attempt a restore from an unrelated host to be sure you really
  have everything needed for restoration.

  Repeat these steps after _all_ configuration changes. Some configuration 
  options are crucial for restoration.

EOF
}

function separator {
  echo "--- $@ ---"
}

function inform {
  echo -e "\nINFO:\n\n$@\n"
}

function warning {
  echo -e "\nWARNING:\n\n$@\n"
}

function warning_oldhome {
  local old=$1 new=$2
  warning " ftplicity changed name to duply since you created your profiles.
  Please rename the old folder
  '$old'
  to
  '$new'
  and this warning will disappear.
  If you decide not to do so profiles will _only_ work from the old location."
}

function error_print {
  echo -e "$@" >&2
}

function error {
  error_print "\nSorry. A fatal ERROR occured:\n\n$@\n"
  exit -1
}

function error_gpg {
  [ -n "$2" ] && local hint="\n  $2\n\n  "
  
  error "$1

Hint${hint:+s}:
  ${hint}Maybe you have not created a gpg key yet (e.g. gpg --gen-key)?
  Don't forget the used _password_ as you will need it.
  When done enter the 8 digit id & the password in the profile conf file.

  The key id can be found doing a 'gpg --list-keys'. In the example output 
  below the key id for the public key would be FFFFFFFF.

  pub   1024D/FFFFFFFF 2007-12-17
  uid                  duplicity
  sub   2048g/899FE27F 2007-12-17
"
}

function error_gpg_test {
  [ -n "$2" ] && local hint="\n  $2\n\n  "

  error "$1

Hint${hint:+s}:
  ${hint}This error means that gpg is probably misconfigured or not working 
  correctly. The error message above should help to solve the problem.
  However, if for some reason $ME_NAME should misinterpret the situation you 
  can define GPG_TEST='disabled' in the conf file to bypass the test.
  Please do not forget to report the bug in order to resolve the problem
  in future versions of $ME_NAME.
"
}

function error_path {
  error "$@
PATH='$PATH'
"
}

function error_to_string {
  [ -n "$1" ] && [ "$1" -eq 0 ] && echo "OK" || echo "FAILED 'code $1'"
}

function duplicity_version_get {
  # use cached value, just print
  var_isset DUPL_VERSION && return

  local DUPL_VERSION_OUT DUPL_VERSION_AWK PYTHON_BIN CMD='duplicity'
  # only run with a user specific python if configured (running by default
  # breaks homebrew as they place a shell wrapper for duplicity in path)
  [ -n "$PYTHON" ] &&\
    CMD="$PYTHON $(qw "$(which duplicity)")"

  DUPL_VERSION_OUT=$($CMD --version)
  DUPL_VERSION=$(echo $DUPL_VERSION_OUT | awk '/^duplicity/{print $2; exit;}')
  #DUPL_VERSION='1.2.3' #'0.7.03' #'0.6.08b' #,0.4.4.RC4,0.6.08b
  DUPL_VERSION_VALUE=0
  DUPL_VERSION_AWK=$(awk -v v="$DUPL_VERSION" 'BEGIN{
  if (match(v,/[^\.0-9]+[0-9]*$/)){
    rest=substr(v,RSTART,RLENGTH);v=substr(v,0,RSTART-1);}
  if (pos=match(rest,/RC([0-9]+)$/)) rc=substr(rest,pos+2)
  split(v,f,"[. ]"); if(f[1]f[2]f[3]~/^[0-9]+$/) vvalue=f[1]*10000+f[2]*100+f[3]; else vvalue=0
  print "#"v"_"rest"("rc"):"f[1]"-"f[2]"-"f[3]
  print "DUPL_VERSION_VALUE=\047"vvalue"\047"
  print "DUPL_VERSION_RC=\047"rc"\047"
  print "DUPL_VERSION_SUFFIX=\047"rest"\047"
  }')
  eval "$DUPL_VERSION_AWK"
  #echo -e ",$DUPL_VERSION,$DUPL_VERSION_VALUE,$DUPL_VERSION_RC,$DUPL_VERSION_SUFFIX,"

  # doublecheck findings and report error
  if [ $DUPL_VERSION_VALUE -eq 0 ]; then
    inform "duplicity version check failed (please report, this is a bug)
the command
  $CMD
resulted in
  $DUPL_VERSION_OUT
"
  elif [ $DUPL_VERSION_VALUE -le 404 ] && [ ${DUPL_VERSION_RC:-4} -lt 4 ]; then
    error "The installed version $DUPL_VERSION is incompatible with $ME_NAME v$ME_VERSION.
You should upgrade your version of $ME_NAME to at least v0.4.4RC4 or
use the older ftplicity version 1.1.1 from $ME_WEBSITE."
  elif [ $DUPL_VERSION_VALUE -le 20100 ] ; then
    error "The installed version $DUPL_VERSION is incompatible with $ME_NAME v$ME_VERSION.
You should upgrade your version of duplicity to at least v2.1.0 or
use the older $ME_NAME version 2.4.3 from $ME_WEBSITE."
  fi
}

function duplicity_version_ge {
  [ "$DUPL_VERSION_VALUE" -ge "$1" ]
}

function duplicity_version_lt {
  ! duplicity_version_ge "$1"
}

# parse interpreter from duplicity shebang
function duplicity_python_binary_parse {
  # reuse cached result
  var_isset 'DUPL_PYTHON_BIN' && {
    [ -n "$DUPL_PYTHON_BIN" ] && {
      echo $DUPL_PYTHON_BIN
      return
    } || return 1
  }

  # no duplicity, nothing to parse
  lookup duplicity || return 1

  local DUPL_BIN=$(which duplicity)
  # test for shebang
  IFS= LC_ALL=C read -rn2 shebang < "$DUPL_BIN" && [ "$shebang" != '#!' ] && {
    DUPL_PYTHON_BIN=""
    return 1
  }

  # parse it or warn
  DUPL_PYTHON_BIN=$(awk 'NR==1&&/^#!/{sub(/^#!( *\/usr\/bin\/env *)?/,""); print}' < "$DUPL_BIN")
  if ! echo "$DUPL_PYTHON_BIN" | grep -q -i 'python'; then
    warning "Could not parse the python interpreter used from duplicity ($DUPL_BIN). Result was 
'$DUPL_PYTHON_BIN'.
"
    DUPL_PYTHON_BIN=""
    return 1
  fi
  
  # success
  echo $DUPL_PYTHON_BIN
  return
}

function run_script { # run pre/post scripts
  local ERR=0
  local SCRIPT="$1"
  if [ ! -z "$PREVIEW" ] ; then
    echo "$([ ! -x "$SCRIPT" ] && echo ". ")$SCRIPT"
  elif [ -r "$SCRIPT" ] ; then 
    echo -n "Running '$SCRIPT' "
    if [ -x "$SCRIPT" ]; then
      OUT=$("$SCRIPT" 2>&1)
      ERR=$?
    else
      OUT=$(. "$SCRIPT" 2>&1)
      ERR=$?
    fi
    [ $ERR -eq "0" ] && echo "- OK" || echo "- FAILED (code $ERR)"
    echo -en ${OUT:+"Output: $OUT\n"} ;
  else
    echo "Skipping n/a script '$SCRIPT'."
  fi
  return $ERR
}

function run_cmd {
  # run or print escaped cmd string
  local CMD_ERR=0
  if [ -n "$PREVIEW" ]; then
    CMD_OUT=$( echo "$@ 2>&1" )
    CMD_MSG="-- Run cmd -- $CMD_MSG --\n$CMD_OUT"
  elif [ -n "$CMD_DISABLED" ]; then
    CMD_MSG="$CMD_MSG (DISABLED) - $CMD_DISABLED"
  else
    echo -n -e "$CMD_MSG"
    CMD_OUT=` eval "$@" 2>&1 `
    CMD_ERR=$?
    if [ "$CMD_ERR" = "0" ]; then
      CMD_MSG=" (OK)"
    else
      CMD_MSG=" (FAILED)"
    fi
  fi
  echo -e "$CMD_MSG"
  # reset
  unset CMD_DISABLED CMD_MSG
  return $CMD_ERR
}

function qw { quotewrap $@; }

function quotewrap {
  local param="$@"
  # quote strings having non word chars (e.g. spaces)
  if echo "$param" | awk '/[^a-zA-Z0-9,\._\+\-@%\/]/{exit 0}{exit 1}'; then
    echo "$param" | awk '{\
      gsub(/[\047]/,"\047\\\047\047",$0);\
      gsub(/[\042]/,"\047\\\042\047",$0);\
      print "\047"$0"\047"}'
    return
  fi
  echo $param
}

function duplicity_params_global {
  # already done? return
  var_isset 'DUPL_PARAMS_GLOBAL' && return
  local DUPL_ARG_ENC

  # use key only if set in config, else leave it to symmetric encryption
  if gpg_disabled; then
    local DUPL_PARAM_ENC='--no-encryption'
  else
    local DUPL_PARAM_ENC=$(gpg_prefix_keyset '--encrypt-key' "${GPG_KEYS_ENC_ARRAY[@]}")
    gpg_signing && local DUPL_PARAM_SIGN=$(gpg_prefix_keyset '--sign-key' "$GPG_KEY_SIGN")
    # interpret password settings
    var_isset 'GPG_PW' && DUPL_ARG_ENC="PASSPHRASE=$(qw "${GPG_PW}")"
    var_isset 'GPG_PW_SIGN' && DUPL_ARG_ENC="${DUPL_ARG_ENC} SIGN_PASSPHRASE=$(qw "${GPG_PW_SIGN}")"
  fi

  local GPG_OPTS=${GPG_OPTS:+"--gpg-options $(qw "${GPG_OPTS}")"}

  # set name for dupl archive folder, since 0.6.0
  if duplicity_version_ge 601; then
    local DUPL_ARCHDIR=''
    if var_isset 'ARCH_DIR'; then
      DUPL_ARCHDIR="--archive-dir $(qw "${ARCH_DIR}")"
      # reuse erronously duply_ prefixed folders from bug #117
      if [ -d "$ARCH_DIR/duply_${PROFILE}" ]; then
        DUPL_ARCHDIR="${DUPL_ARCHDIR} --name $(qw "duply_${PROFILE}")"
      else
        DUPL_ARCHDIR="${DUPL_ARCHDIR} --name $(qw "${PROFILE}")"
      fi
    else
      DUPL_ARCHDIR="--name $(qw "duply_${PROFILE}")"
    fi
  fi

DUPL_PARAMS_GLOBAL="${DUPL_ARCHDIR} ${DUPL_PARAM_ENC}\
 ${DUPL_PARAM_SIGN} ${VERBOSITY:+--verbosity $VERBOSITY}\
 ${GPG_OPTS}"

DUPL_VARS_GLOBAL="TMPDIR='$TEMP_DIR'\
 ${DUPL_ARG_ENC}"
}

# function to filter the DUPL_PARAMS var from user conf
function duplicity_params_conf {
  local OUT="$DUPL_PARAMS"
  # reuse cmd var from main loop
  ## in/exclude parameters are currently not supported on 
  ## cleanup, status (collection_status), list (list_current_files), purge* (remove_*), fetch/restore
  case $cmd in
    cleanup | status | list | purge* | restore | fetch )
      # filter exclude params from fetch/restore/status
      OUT=$(eval stripXcludes $OUT)
    ;;
  esac

  case $cmd in
    bkp | incr | full )
      # nothing to strip, we're backing up'
    ;;
    *)
      OUT=$(eval stripBkpOnlyParams $OUT)
    ;;
  esac

  # print result
  echo "$OUT"
}

# strip in/exclude parameters from param string
function stripXcludes {
  local STRIPNEXT OUT;
  for p in "$@"; do
    #echo "calc =$p="
    if [ -n "$STRIPNEXT" ]; then
      unset STRIPNEXT
      # strip the value of previous parameter
      continue
    elif echo "$p" | awk '/^\-\-(in|ex)clude(\-[a-zA-Z\-]+)?$/{exit 0;}{exit 1;}'; then
      # strips e.g. --include /foo/bar
      STRIPNEXT="yes"
      continue
    elif echo "$p" | awk '/^\-\-(in|ex)clude(\-[a-zA-Z\-]+)?=/{exit 0;}{exit 1;}'; then
      # strips e.g. --include=/foo/bar
      continue
    fi

    OUT="$OUT $(qw "$p")"
  done
  echo "$OUT"
}

# strip backup only parameters from param string
function stripBkpOnlyParams {
  local STRIPNEXT OUT;

  for p in "$@"; do
    if [ -n "$STRIPNEXT" ]; then
      unset STRIPNEXT
      # strip the value of previous parameter
      continue
    elif echo "$p" | awk '/^\-\-(allow-source-mismatch|asynchronous-upload|dry-run)$/{exit 0;}{exit 1;}'; then
      continue
    elif echo "$p" | awk '/^\-\-(volsize)$/{exit 0;}{exit 1;}'; then
      # strips e.g. --volsize 100
      STRIPNEXT="yes"
      continue
    elif echo "$p" | awk '/^\-\-volsize=/{exit 0;}{exit 1;}'; then
      # strips e.g. --volsize=100
      continue
    fi

    OUT="$OUT $(qw "$p")"
  done
  echo "$OUT"
}

function duplify { # the actual wrapper function
  local PARAMSNOW DUPL_CMD DUPL_CMD_PARAMS

  # put command (with params) first in duplicity parameters
  for param in "$@" ; do
  # split cmd from params (everything before splitchar --)
    if [ "$param" == "--" ] ; then
      PARAMSNOW=1
    else
      # wrap in quotes to protect from spaces
      [ ! $PARAMSNOW ] && \
        DUPL_CMD="$DUPL_CMD $(qw "$param")" \
      || \
        DUPL_CMD_PARAMS="$DUPL_CMD_PARAMS $(qw "$param")"
    fi
  done

  # init global duplicity parameters same for all tasks
  duplicity_params_global

  local RUN=eval CMD=duplicity
  # run in cmd line preview mode if requested
  var_isset 'PREVIEW' && RUN=echo
  # only run with a user specific python if configured (running by default
  # breaks homebrew as they place a shell wrapper for duplicity in path)
  # resolve duplicity path for usage with python interpreter
  [ -n "$PYTHON" ] &&\
    CMD="$PYTHON $(qw "$(which duplicity)")"

$RUN "${DUPL_VARS_GLOBAL} ${BACKEND_PARAMS}\
 ${DUPL_PRECMD} $CMD $DUPL_CMD $DUPL_PARAMS_GLOBAL $(duplicity_params_conf)\
 $GPG_USEAGENT $(gpg_custom_binary) $DUPL_CMD_PARAMS"

  local ERR=$?
  return $ERR
}

function secureconf { # secure the configuration dir
  #PERMS=$(ls -la $(dirname $CONFDIR) | grep -e " $(basename $CONFDIR)\$" | awk '{print $1}')
  local PERMS="$(ls -la "$CONFDIR/." | awk 'NR==2{print $1}')"
  if [ "${PERMS/#drwx------*/OK}" != 'OK' ] ; then
    chmod u+rwX,go= "$CONFDIR"; local ERR=$?
    warning "The profile's folder 
'$CONFDIR'
permissions are not safe ($PERMS). Secure them now. - ($(error_to_string $ERR))"
  fi
}

# params are $1=timeformatstring (default like date output), $2=epoch seconds since 1.1.1970 (default now)
function date_fix {
  local DEFAULTFORMAT='%a %b %d %H:%M:%S %Z %Y'
  local date
  #[ "$1" == "%N" ] && return #test the no nsec test below
  # gnu date with -d @epoch
  date=$(date ${2:+-d @$2} ${1:++"$1"} 2> /dev/null) && \
    echo $date && return
  # date bsd,osx with -r epoch
  date=$(date ${2:+-r $2} ${1:++"$1"} 2> /dev/null) && \
    echo $date && return
  # date busybox with -d epoch -D %s
  date=$(date ${2:+-d $2 -D %s} ${1:++"$1"} 2> /dev/null) && \
    echo $date && return
  ## some date commands do not support giving a time w/o setting it systemwide (irix,solaris,others?)
  # python fallback
  #date=$("$(python_binary)" -c "import time;print time.strftime('${1:-$DEFAULTFORMAT}',time.localtime(${2}))" 2> /dev/null) && \
  #  echo $date && return
  # awk fallback
  date=$(awk "BEGIN{print strftime(\"${1:-$DEFAULTFORMAT}\"${2:+,$2})}" 2> /dev/null) && \
    echo $date && return
  # perl fallback
  date=$(perl  -e "use POSIX qw(strftime);\$date = strftime(\"${1:-$DEFAULTFORMAT}\",localtime(${2}));print \"\$date\n\";" 2> /dev/null) && \
    echo $date && return
  # error
  echo "ERROR"
  return 1
}

function nsecs {
  local NSECS
  # test if date supports nanosecond output
  if ! var_isset NSECS_DISABLED; then
    NSECS=$(date_fix %N 2> /dev/null | head -1 |grep -e "^[[:digit:]]\{9\}$")
    [ -n "$NSECS" ] && NSECS_DISABLED=0 || NSECS_DISABLED=1
  fi

  # add 9 digits, not all date(s) deliver nsecs e.g. busybox date
  if [ "$NSECS_DISABLED" == "1" ]; then
    date_fix %s000000000
  else
    date_fix %s%N
  fi
}

function nsecs_to_sec {
  echo $(($1/1000000000)).$(printf "%03d" $(($1/1000000%1000)) )
}

function datefull_from_nsecs {
  date_from_nsecs $1 '%F %T'
}

function date_from_nsecs {
  local FORMAT=${2:-%T}
  local TIME=$(nsecs_to_sec $1)
  local SECS=${TIME%.*}
  local DATE=$(date_fix "${FORMAT}" ${SECS:-0})
  echo $DATE.${TIME#*.}
}

function var_isset {
  if [ -z "$1" ]; then
    echo "ERROR: function var_isset needs a string as parameter"
  elif eval "[ \"\${$1}\" == 'not_set' ]" || eval "[ \"\${$1-not_set}\" != 'not_set' ]"; then
    return 0
  fi
  return 1
}

function is_condition {
  local CMD=$(tolower "$1")
  [ "$CMD" == 'and' ] || [ "$CMD" == 'or' ]
}

function is_groupMarker {
  local CMD=$(tolower "$1")
  [ "$CMD" == 'groupin' ] || [ "$CMD" == 'groupout' ]
}

function is_command {
  local CMD=$(tolower "$1")
  ! is_condition "$CMD" && ! is_groupMarker "$CMD"
}

function url_encode {
  # utilize python, silently do nothing on error - because no python no duplicity
  local PYTHON_RUNNER
  PYTHON_RUNNER="$(duplicity_python_binary_parse)" ||\
  PYTHON_RUNNER="python" &&\
    OUT=$($PYTHON_RUNNER -c "
try: import urllib.request as urllib
except ImportError: import urllib
print(urllib.${2}quote('$1'));
" 2>/dev/null ); ERR=$?
  [ "$ERR" -eq 0 ] && echo $OUT || echo $1
}

function url_decode {
  # reuse function above with a simple string param hack
  url_encode "$1" "un"
}

function toupper {
  echo "$@"|awk '$0=toupper($0)'
}

function tolower {
  echo "$@"|awk '$0=tolower($0)'
}

function isnumber {
  case "$*" in
    ''|*[!0-9]*) return 1;;
    *) return 0;;
  esac
}

#function tmp_space {
#  
#  if ! isnumber $VOLSIZE; then
#    inform "failed to determine free space (please report, this is a bug)"
#    return
#  fi
#  
# get free temp space
#  TEMP_FREE="$(df -P -k "$TEMP_DIR" 2>/dev/null | awk 'END{pos=(NF-2);if(pos>0) print $pos;}')"
#  # check for free space or FAIL
#  if [ $((${TEMP_FREE:-0}-${VOLSIZE:-0}*1024)) -lt 0-lt 0 ]; then
#    error "Temporary file space '$TEMP_DIR' free space is smaller ($((TEMP_FREE/1024))MB)
#than one duplicity volume (${VOLSIZE}MB).
#    
#  Hint: Free space or change TEMP_DIR setting."
#fi
#
#}

function gpg_disabled {
  echo "${GPG_KEY}" | grep -iq -e '^disabled$'
}

# usage: join SEPARATOR "entry1" "entry2"
function join {
  local SEP="$1" ENTRY OUT; shift;
  for ENTRY in "$@"; do
    ENTRY=${ENTRY//$SEP/\\$SEP}
    [ -z "$OUT" ] && OUT=$ENTRY || OUT="$OUT$SEP$ENTRY"
  done
  echo $OUT
}

function gpg_testing {
  [ "$GPG_TEST" != "disabled" ]
}

function gpg_signing {
  echo ${GPG_KEY_SIGN} | grep -v -q -e '^disabled$'
}

function gpg_keytype {
  echo "$1" | awk '/^PUB$/{print "public"}/^SEC$/{print "secret"}'
}

# parameter key id, key_type
function gpg_keyfile {
  local GPG_KEY=$(gpg_key_legalize $1) TYPE="$2"
  local KEYFILE="${KEYFILE//.asc/${GPG_KEY:+.$GPG_KEY}.asc}"
  echo "${KEYFILE//.asc/${TYPE:+.$(tolower $TYPE)}.asc}"
}

# parameter key id
function gpg_import {
  local i FILE FOUND=0 KEY_ID="$1" KEY_TYPE="$2" KEY_FP="" ERR=0
  [ "$GPG_IMPORT" = "disabled" ] && {
    echo "Skipping import of needed $(gpg_keytype "$KEY_TYPE") key '$KEY_ID'. (GPG_IMPORT='disabled')"
    return
  }

  # create a list of legacy key file names and current naming scheme
  # we always import pub and sec if they are avail in conf folder
  local KEYFILES=( "$CONFDIR/gpgkey" $(gpg_keyfile "$KEY_ID") \
                   $(gpg_keyfile "$KEY_ID" "$KEY_TYPE") )

  # Try autoimport from existing old gpgkey files 
  # and new gpgkey.XXX.asc files (since v1.4.2)
  # and even newer gpgkey.XXX.[pub|sec].asc
  for (( i = 0 ; i < ${#KEYFILES[@]} ; i++ )); do
    FILE=${KEYFILES[$i]}
    if [ -f "$FILE" ]; then
      FOUND=1
      
      CMD_MSG="Import keyfile '$FILE' to keyring"
      run_cmd gpg $GPG_OPTS --batch --import $(qw "$FILE")
      if [ "$?" != "0" ]; then 
        warning "Import failed.${CMD_OUT:+\n$CMD_OUT}"
        ERR=1
        # continue with next
        continue
      fi
    fi
  done

  if [ "$FOUND" -eq 0 ]; then
    echo "Notice: No keyfile for '$KEY_ID' found in profile folder."
    return 1
  fi

  # try to set trust automagically
  CMD_MSG="Autoset trust of key '$KEY_ID' to ultimate"
  run_cmd echo $(gpg_fingerprint "$KEY_ID"):6: \| gpg $GPG_OPTS --import-ownertrust --batch --logger-fd 1
  if [ "$?" = "0" ] && [ -z "$PREVIEW" ]; then 
   # success on all levels, we're done
   return $ERR
  fi

  # failover: user has to set trust manually
  echo -e "For $ME_NAME to work you have to set the trust level 
with the command \"trust\" to \"ultimate\" (5) now.
Exit the edit mode of gpg with \"quit\"."
  CMD_MSG="Running gpg to manually edit key '$KEY_ID'"
  run_cmd sleep 5\; gpg $GPG_OPTS --edit-key $(qw "$KEY_ID")

  return $ERR
}

# see 'How to specify a user ID' on gpg manpage
function gpg_fingerprint {
  gpg $GPG_OPTS --fingerprint "$1" 2>&1 | \
  awk 'NR==2{sub(/^.*=/,"");gsub(/[ \t]/,""); if ( $0 !~ /^[A-F0-9]+$/ || length($0) != 40 ) exit 1; print}'
}

function gpg_export_if_needed {
  [ "$GPG_EXPORT" = 'disabled' ] && { \
    echo "Skipping export of gpg keys. (GPG_EXPORT='disabled')"
    return
  }

  local SUCCESS FILE KEY_TYPE
  local TMPFILE="$TEMP_DIR/${ME_NAME}.$$.$(date_fix %s).gpgexp"
  for KEY_ID in "$@"; do
    # check if already exported, do it if not
    for KEY_TYPE in PUB SEC; do
      FILE="$(gpg_keyfile "$KEY_ID" $KEY_TYPE)"
      if [ ! -f "$FILE" ] && eval gpg_$(tolower $KEY_TYPE)_avail \"$KEY_ID\"; then

        # exporting
        CMD_MSG="Backup $(gpg_keytype "$KEY_TYPE") key '$KEY_ID' to profile."
        # gpg2.1 insists on passphrase here, gpg2.0- happily exports w/o it
        # we pipe an empty string when GPG_PW is not set to avoid gpg silently waiting for input
        run_cmd $(gpg_pass_pipein GPG_PW_SIGN GPG_PW) gpg $GPG_OPTS $GPG_USEAGENT $(gpg_param_passwd GPG_PW_SIGN GPG_PW) --armor --export"$(test "SEC" = "$KEY_TYPE" && echo -secret-keys)" $(qw "$KEY_ID") '>>' $(qw "$TMPFILE")
        CMD_ERR=$?

        if [ "$CMD_ERR" = "0" ]; then
          CMD_MSG="Write file '"$(basename "$FILE")"'"
          run_cmd mv $(qw "$TMPFILE") $(qw "$FILE")
        fi

        if [ "$CMD_ERR" != "0" ]; then
          warning "Backup failed.${CMD_OUT:+\n$CMD_OUT}"
        else
          SUCCESS=1
        fi

        # cleanup
        rm $(qw "$TMPFILE") 1>/dev/null 2>&1
      fi
    done
  done
  
  [ -n "$SUCCESS" ] && inform "$ME_NAME exported new keys to your profile.
You should backup your changed profile folder now and store it in a safe place."
}

# replace all non-alnum chars with underscore (for file operations)
function gpg_key_legalize {
  echo $* | awk '{gsub(/[^a-zA-Z0-9]/,"_",$0); print}'
}

function gpg_key_cache {
  local RES
  local MODE=$1
  shift
  local PREFIX="GPG_KEY"
  local SUFFIX=$(gpg_key_legalize "$@")
  local KEYID="$*"
  local CACHE="${PREFIX}_${MODE}_${SUFFIX}"
  if [ "$MODE" = "RESET" ]; then
    eval unset ${PREFIX}_PUB_$SUFFIX ${PREFIX}_SEC_$SUFFIX
    return 255
  elif ! var_isset "$CACHE"; then
    if [ "$MODE" = "PUB" ]; then
      RES=$(gpg $GPG_OPTS --list-key "$KEYID" > /dev/null 2>&1; echo -n $?)
    elif [ "$MODE" = "SEC" ]; then
      RES=$(gpg $GPG_OPTS --list-secret-key "$KEYID" > /dev/null 2>&1; echo -n $?)
    else
      return 255
    fi
    eval $CACHE=$RES
  fi
  eval return \$$CACHE
}

function gpg_pub_avail {
  gpg_key_cache PUB "$@"
}

function gpg_sec_avail {
  gpg_key_cache SEC "$@"
}

function gpg_key_format {
  echo $1 | grep -q '^[0-9a-fA-F]\{8\}$'
}

# splits a comma separated line into lines, respects escaped commas
function gpg_split_keyset {
  local LIST
  LIST=$(echo "$@" | awk '{ gsub(/,/,"\n",$0); gsub(/\\\n/,",",$0); print $0 }')
  echo -e "$LIST"
}

function gpg_prefix_keyset {
  local PREFIX="$1" OUT=""
  shift
  for KEY_ID in "$@"; do
    OUT="${OUT} $PREFIX $(qw ${KEY_ID})"
  done
  echo $OUT
}

# grep a variable from conf text file (currently not used)
function gpg_passwd {
  [ -r "$CONF" ] && \
  awk '/^[ \t]*GPG_PW[ \t=]/{\
        sub(/^[ \t]*GPG_PW[ \t]*=*/,"",$0);\
        gsub(/^[ \t]*[\047"]|[\047"][ \t]*$/,"",$0);\
        print $0; exit}' "$CONF"
}

# return success if at least one secret key is available
function gpg_key_decryptable {
  # no keys, no problem
  gpg_symmetric && return 0

  local KEY_ID
  for KEY_ID in "${GPG_KEYS_ENC_ARRAY[@]}"; do
    gpg_sec_avail "$KEY_ID" && return 0
  done
  return 1
}

function gpg_symmetric {
  [ -z "${GPG_KEY}${GPG_KEYS_ENC_ARRAY}" ]
}

# checks for max two params if they are set, typically GPG_PW & GPG_PW_SIGN
function gpg_param_passwd {
  var_isset GPG_USEAGENT && exit 1
  
  if ( [ -n "$1" ] && var_isset "$1" ) || ( [ -n "$2" ] && var_isset "$2" ); then
    echo "--passphrase-fd 0 --batch"
  fi
}

# select the earliest defined and create an "echo <value> |" string
function gpg_pass_pipein {
  var_isset GPG_USEAGENT && exit 1
  
  for var in "$@"
  do
    if var_isset "$var"; then
      echo "echo $(qw $(eval echo \$$var)) |"
      return 0
    fi
  done
  
  return 1
}

# checks if gpg-agent is available, returns error code
# 0 on success
# 1 if GPG_AGENT_INFO is not set (unused, should probably be merged w/ 3)
# 2 if GPG_AGENT_INFO is stale
# 3 cannot connect to gpg-agent
function gpg_agent_avail {
  # GPG_AGENT_INFO is deprecated in gpg2.1, 
  # first try to connect to a possibly running agent here
  local ERR=3
  gpg-agent > /dev/null 2>&1 && return 0

  # detect stale pid in legacy GPG_AGENT_INFO env var
  if var_isset GPG_AGENT_INFO; then
    # check if a pid matching process is running at all
    local GPG_AGENT_PID=$(echo $GPG_AGENT_INFO|awk -F: '{print $2}')
    if isnumber "$GPG_AGENT_PID"; then
      ps -p "$GPG_AGENT_PID" > /dev/null 2>&1 || ERR=2
    fi
  fi

  return $ERR
}

function gpg_custom_binary {
  var_isset GPG && [ "$GPG" != "$DEFAULT_GPG" ] &&\
    echo "--gpg-binary $(qw "$GPG")"
}

function gpg_binary {
  local BIN
  var_isset GPG && BIN="$GPG" || BIN="$DEFAULT_GPG"
  echo "$BIN"
}

function gpg_avail {
  lookup "$(gpg_binary)"
}

# enforce the use of our selected gpg binary
function gpg {
  command "$(gpg_binary)" "$@"
}
export -f gpg

# start of script #######################################################################

# confidentiality first, all we create is only readable by us
umask 077

# check if ftplicity is there & executable
[ -n "$ME_LONG" ] && [ -x "$ME_LONG" ] || error "$ME missing. Executable & available in path? ($ME_LONG)"

if [ ${#@} -eq 1 ]; then
  cmd="${1}"
else
  FTPLCFG="${1}" ; cmd="${2}"
fi

# deal with command before profile validation calls
# show requested version
# OR requested usage info
# OR create a profile
# OR fall through
##if [ ${#@} -le 2 ]; then
case "$cmd" in
  changelog)
    changelog
    exit 0
    ;;
  create)
    set_config
    if [ -d "$CONFDIR" ]; then
      error "The profile '$FTPLCFG' already exists in
'$CONFDIR'.

Hint:
 If you _really_ want to create a new profile by this name you will 
 have to manually delete the existing profile folder first."
      exit 1
    else
      create_config
      exit 0
    fi
    ;;
  txt2man)
    set_config
    usage_txt2man
    exit 0
    ;;
  usage|-help|--help|-h|-H)
    set_config
    usage_info
    exit 0
    ;;
  version|-version|--version|-v|-V)
    # profile can override GPG/PYTHON, so import it if it was given
    var_isset FTPLCFG && {
      set_config
      [ -r "$CONF" ] && . "$CONF" || warning "Cannot import config '$CONF'."
    }
    version_info_using
    exit 0
    ;;
  # fallthrough.. we got a command that needs an existing profile
  *)
    # if we reach here, user either forgot profile or chose wrong profileless command
    if [ ${#@} -le 1 ]; then
      error "\
 Missing or wrong parameters. 
 Only the commands 
   changelog, create, usage, txt2man, version
 can be called without selecting an existing profile first.
 Your command was '$cmd'.

 Hint: Run '$ME usage' to get help."
    fi
esac


# Hello world
echo "Start $ME v$ME_VERSION, time is $(date_fix '%F %T')."

# check system environment

# is duplicity avail
lookup duplicity || error_path "duplicity missing. installed und available in path?"
# init, exec duplicity version check info
duplicity_version_get

# check for certain important helper programs
# TODO: we should probably check for duplicity and $PYTHON (if set) here too
for f in awk grep ; do
  lookup "$f" || \
    error_path "$f missing. installed und available in path?"
done

### read configuration
set_config
# check validity
if [ ! -d "$CONFDIR" ]; then 
    error "Selected profile '$FTPLCFG' does not resolve to a profile folder in
'$CONFDIR'.

Hints:
Select one of the available profiles: $(for d in "$(dirname "$CONFDIR")"/*/; do [ -e "$d" ] || [ -L "$d" ] || continue; printf "$sep'$(basename "$d")'"; sep=",";done)
 Use '$ME <name> create' to create a new profile.
 Use '$ME usage' to get usage help."
elif [ ! -x "$CONFDIR" ]; then
    error "\
Profile folder in '$CONFDIR' cannot be accessed.

Hint: 
 Check the filesystem permissions and set directory accessible e.g. 'chmod 700'."
elif [ ! -f "$CONF" ] ; then
  error "'$CONF' not found."
elif [ ! -r "$CONF" ] ; then
  error "'$CONF' not readable."
else
  . "$CONF"
  #KEYFILE="${KEYFILE//.asc/${GPG_KEY:+.$GPG_KEY}.asc}"
  TEMP_DIR=${TEMP_DIR:-'/tmp'}
  # backward compatibility: old TARGET_PW overrides silently new TARGET_PASS if set
  if var_isset 'TARGET_PW'; then
    TARGET_PASS="${TARGET_PW}"
  fi
fi
echo "Using profile '$CONFDIR'."

# secure config dir, if needed w/ warning
secureconf

# split TARGET in handy variables
TARGET_SPLIT_URL=$(echo "$TARGET" | awk '{ \
  target=$0; match(target,/^([^\/:]+):\/\//); \
  prot=substr(target,RSTART,RLENGTH);\
  rest=substr(target,RSTART+RLENGTH); \
  if (credsavail=match(rest,/^[^@]*@/)){\
    creds=substr(rest,RSTART,RLENGTH-1);\
    credcount=split(creds,cred,":");\
    rest=substr(rest,RLENGTH+1);\
    # split creds with regexp\
    match(creds,/^([^:]+)/);\
    user=substr(creds,RSTART,RLENGTH);\
    pass=substr(creds,RSTART+1+RLENGTH);\
  };\
  # filter quotes or escape them\
  gsub(/[\047\042]/,"",prot);\
  gsub(/[\047\042]/,"",rest);\
  gsub(/[\047]/,"\047\\\047\047",creds);\
  print "TARGET_URL_PROT=\047"prot"\047\n"\
         "TARGET_URL_HOSTPATH=\047"rest"\047\n"\
         "TARGET_URL_CREDS=\047"creds"\047\n";\
   if(user){\
     gsub(/[\047]/,"\047\\\047\047",user);\
     print "TARGET_URL_USER=\047"user"\047\n"}\
   if(pass){\
     gsub(/[\047]/,"\047\\\047\047",pass);\
     print "TARGET_URL_PASS=$(url_decode \047"pass"\047)\n"}\
  }')
eval "${TARGET_SPLIT_URL}"

# fetch commmand from parameters ########################################################
# Hint: cmds is also used to check if authentification info sufficient in the next step 
cmds="$2"; shift 2

# complain if command(s) missing
[ -z $cmds ] && error "  No command given.

  Hint: 
    Use '$ME usage' to get usage help."

# process params
for param in "$@"; do
  #echo !$param!
  case "$param" in
    # enable ftplicity preview mode
    '--preview')
      PREVIEW=1
      ;;
    # interpret duplicity disable encr switch
    '--disable-encryption')
      GPG_KEY='disabled'
      ;;
    *)
      if [ `echo "$param" | grep -e "^-"` ] || \
         [ `echo "$last_param" | grep -e "^-"` ] ; then
        # forward parameter[/option pairs] to duplicity
        dupl_opts["${#dupl_opts[@]}"]=${param}
      else
        # anything else must be a parameter (e.g. for fetch, ...)
        ftpl_pars["${#ftpl_pars[@]}"]=${param}
      fi
      last_param=${param}
      ;;
  esac
done

# plausibility check config - VARS & KEY ################################################
# check if src, trg, trg pw
# auth info sufficient 
# gpg key, gpg pwd (might be empty) set in config
# OR key in local gpg db
# OR key can be imported from keyfile 
# OR fail
if [ -z "$SOURCE" ] || [ "$SOURCE" == "${DEFAULT_SOURCE}" ]; then
 error " Source Path (setting SOURCE) not set or still default value in conf file 
 '$CONF'."

elif [ -z "$TARGET" ] || [ "$TARGET" == "${DEFAULT_TARGET}" ]; then
 error " Backup Target (setting TARGET) not set or still default value in conf file 
 '$CONF'."

elif var_isset 'TARGET_USER' && var_isset 'TARGET_URL_USER' && \
     [ "${TARGET_USER}" != "${TARGET_URL_USER}" ]; then
 error " TARGET_USER ('${TARGET_USER}') _and_ user in TARGET url ('${TARGET_URL_USER}') 
 are configured with different values. There can be only one.
 
 Hint: Remove conflicting setting."

elif var_isset 'TARGET_PASS' && var_isset 'TARGET_URL_PASS' && \
     [ "${TARGET_PASS}" != "${TARGET_URL_PASS}" ]; then
 error " TARGET_PASS ('${TARGET_PASS}') _and_ password in TARGET url ('${TARGET_URL_PASS}') 
 are configured with different values. There can be only one.
 
 Hint: Remove conflicting setting."
fi

# GPG config plausibility check1 (disabled check) #############################
if gpg_disabled; then
  : # encryption disabled, all is well
elif [ -z "${GPG_KEY}${GPG_KEYS_ENC}${GPG_KEY_SIGN}" ] && ! var_isset 'GPG_PW'; then
  warning "GPG_KEY, GPG_KEYS_ENC, GPG_KEY_SIGN and GPG_PW are empty/not set in conf file 
'$CONF'.
Will disable encryption for duplicity now.

Hint: 
 If you really want to use _no_ encryption you can disable this warning by 
 setting GPG_KEY='disabled' in conf file."
  GPG_KEY='disabled'
fi

# GPG availability check (now we know if gpg is really needed)#################
if ! gpg_disabled; then 
  gpg_avail || error_path "gpg '$(gpg_binary)' missing. installed und available in path?"
fi

# Output versions info ########################################################
using_info

# GPG create key settings, config check2 (needs gpg) ##########################
if gpg_disabled; then
  : # the following tests are not necessary
else

# we test this early as any invocation gpg2.1+ starts gpg-agent automatically
GPG_AGENT_ERR=$(gpg_agent_avail ; echo $?)

# enc key still default?
if [ "$GPG_KEY" == "${DEFAULT_GPG_KEY}" ]; then 
  error_gpg "Encryption Key GPG_KEY still default in conf file 
'$CONF'."
fi

# create array of gpg encr keys, for further processing
OIFS="$IFS" IFS=$'\n'
GPG_KEYS_ENC_ARRAY=( $( gpg_split_keyset ${GPG_KEY},${GPG_KEYS_ENC} ) )
IFS="$OIFS"

# pw set? 
# symmetric needs one, always
if gpg_symmetric && ( [ -z "$GPG_PW" ] || [ "$GPG_PW" == "${DEFAULT_GPG_PW}" ] ) \
  ; then
  error_gpg "Encryption passphrase GPG_PW (needed for symmetric encryption) 
is empty/not set or still default value in conf file 
'$CONF'."
fi
# this is a technicality, we can only pump one pass via pipe into gpg
# but symmetric already always needs one for encryption
if gpg_symmetric && var_isset GPG_PW && var_isset GPG_PW_SIGN &&\
  [ -n "$GPG_PW_SIGN" ] && [ "$GPG_PW" != "$GPG_PW_SIGN" ]; then
  error_gpg "GPG_PW _and_ GPG_PW_SIGN are defined but not identical in config
'$CONF'.
This is unfortunately impossible. For details see duplicity manpage, 
section 'A Note On Symmetric Encryption And Signing'.

Tip: Separate signing keys may have empty passwords e.g. GPG_PW_SIGN=''.
Tip2: Use gpg-agent."
fi

# test - GPG KEY AVAILABILITY ##################################################

# check gpg public keys availability, try import if needed
for (( i = 0 ; i < ${#GPG_KEYS_ENC_ARRAY[@]} ; i++ )); do
  KEY_ID="${GPG_KEYS_ENC_ARRAY[$i]}"
  # test availability, try to import, retest
  if ! gpg_pub_avail "${KEY_ID}"; then
    echo "Encryption public key '${KEY_ID}' not in keychain. Try to import from profile."
    gpg_import "${KEY_ID}" PUB
    gpg_key_cache RESET "${KEY_ID}"
    gpg_pub_avail "${KEY_ID}" || { \
      gpg_testing && error_gpg \
      "Needed public gpg key '${KEY_ID}' is not available in keychain." \
      "Doublecheck if the above key is listed by 'gpg --list-keys' or available
  as gpg key file '$(basename "$(gpg_keyfile "${KEY_ID}")")' in the profile folder.
  If not you can put it there and $ME_NAME will autoimport it on the next run.
  Alternatively import it manually as the user you plan to run $ME_NAME with."
    }
  else
    echo "Public key '${KEY_ID}' found in keychain."
  fi
done

# check gpg encr secret encryption keys availability and fail
# if none is available after a round of importing trials
gpg_key_decryptable || \
{
  echo "Missing secret keys for decryption in keychain."
  for (( i = 0 ; i < ${#GPG_KEYS_ENC_ARRAY[@]} ; i++ )); do
    KEY_ID="${GPG_KEYS_ENC_ARRAY[$i]}"
    # test availability, try to import, retest
    if ! gpg_sec_avail "${KEY_ID}"; then
    echo "Try to import secret key '${KEY_ID}' from profile."
      gpg_import "${KEY_ID}" SEC
      gpg_key_cache RESET "${KEY_ID}"
    fi
  done
  gpg_key_decryptable || \
  {
    gpg_testing && error_gpg_test "None of the configured keys '$(join "','" "${GPG_KEYS_ENC_ARRAY[@]}")' \
has a secret key in the keychain. Decryption will be impossible!"
  }
}

# gpg secret sign key availability
# if none set, autoset first encryption key as sign key
if ! gpg_signing; then
  echo "Signing disabled per configuration."
# try first key, if one set
elif ! var_isset 'GPG_KEY_SIGN'; then
  KEY_ID="${GPG_KEYS_ENC_ARRAY[0]}"
  if [ -z "${KEY_ID}" ]; then
    echo "Signing disabled. No GPG_KEY entries in config."
    GPG_KEY_SIGN='disabled'
  else  
    # use avail OR try import OR fail
    if gpg_sec_avail "${KEY_ID}"; then
      GPG_KEY_SIGN="${KEY_ID}"
    else
      echo "Signing secret key '${KEY_ID}' not found."
      gpg_import "${KEY_ID}" SEC
      gpg_key_cache RESET "${KEY_ID}"
      if gpg_sec_avail "${KEY_ID}"; then
        GPG_KEY_SIGN="${KEY_ID}"
      fi
    fi

    # interpret sign key setting
    if var_isset 'GPG_KEY_SIGN'; then
      echo "Autoset found secret key of first GPG_KEY entry '${KEY_ID}' for signing."
    else
      echo "Signing disabled. First GPG_KEY entry's '${KEY_ID}' private key is missing."
      GPG_KEY_SIGN='disabled'
    fi
  fi
else
  KEY_ID="${GPG_KEY_SIGN}"
  if ! gpg_sec_avail "${KEY_ID}"; then
    inform "Secret signing key defined in setting GPG_KEY_SIGN='${KEY_ID}' not found.\nTry to import."
    gpg_import "${KEY_ID}" SEC
    gpg_key_cache RESET "${KEY_ID}"
    gpg_sec_avail "${KEY_ID}" || error_gpg_key "${KEY_ID}" "Private"
  fi
fi

# using GPG_AGENT_ERR set early above, try to autoenable gpg-agent or issue some warnings
# key enc can deal without, but might profit from gpg-agent
# if GPG_PW is not set alltogether
# if signing key is different from first (main) enc key (we can only pipe one pass into gpg)
if ! gpg_symmetric && \
   ( ! var_isset GPG_PW || \
     ( gpg_signing && ! var_isset GPG_PW_SIGN && [ "$GPG_KEY_SIGN" != "${GPG_KEYS_ENC_ARRAY[0]}" ] ) ); then

  if [ "$GPG_AGENT_ERR" -eq 1 ]; then
    warning "Cannot use gpg-agent. GPG_AGENT_INFO not set."
  elif [ "$GPG_AGENT_ERR" -eq 2 ]; then
    warning "Cannot use gpg-agent! GPG_AGENT_INFO contains stale pid."
  elif [ "$GPG_AGENT_ERR" -eq 3 ]; then
    warning "No running gpg-agent found although GPG_PW or GPG_PW_SIGN (enc != sign key) not set."
  else
    echo "Enable gpg-agent usage. Running gpg-agent instance found and GPG_PW or GPG_PW_SIGN (enc != sign key) not set."
    GPG_USEAGENT="--use-agent"
  fi
fi

# end GPG config plausibility check2 
fi

# config plausibility check - SPACE ###########################################

# is tmp is a folder and writable
CMD_MSG="Checking TEMP_DIR '${TEMP_DIR}' is a folder and writable"
run_cmd test -d $(qw "$TEMP_DIR") '&&' test -w $(qw "$TEMP_DIR")
if [ "$?" != "0" ]; then
    error "Temporary file space '$TEMP_DIR' is not a directory or writable."
fi


# get volsize, default duplicity volume size is 25MB since v0.5.07
#VOLSIZE=${VOLSIZE:-25}
# double if asynch is on
#echo $@ $DUPL_PARAMS | grep -q -e '--asynchronous-upload' && FACTOR=2 || FACTOR=1

# TODO: check for enough (async= upload space and WARN only
#       use function tmp_space 
#echo TODO: reimplent tmp space check


# test - GPG SANITY #####################################################################
# if encryption is disabled, skip this whole section
if gpg_disabled; then
  echo -e "Test - En/Decryption skipped. (GPG='disabled')"
elif ! gpg_testing; then 
  echo -e "Test - En/Decryption skipped. (GPG_TEST='disabled')"
else

GPG_TEST_PREFIX="$TEMP_DIR/${ME_NAME}.$$.$(date_fix %s)"
function cleanup_gpgtest { 
  echo -en "Cleanup - Delete '${GPG_TEST_PREFIX}_*'"
  rm "${GPG_TEST_PREFIX}"_* 2>/dev/null && echo "(OK)" || echo "(FAILED)"
}

# signing enabled?
if gpg_signing; then
  CMD_PARAM_SIGN="--sign --default-key $(qw ${GPG_KEY_SIGN})"
  CMD_MSG_SIGN="Sign with '${GPG_KEY_SIGN}'"
fi

# using keys
if [ ${#GPG_KEYS_ENC_ARRAY[@]} -gt 0 ]; then

  for KEY_ID in "${GPG_KEYS_ENC_ARRAY[@]}"; do
    CMD_PARAMS="$CMD_PARAMS -r $(qw ${KEY_ID})"
  done
  # check encrypting
  CMD_MSG="Test - Encrypt to '$(join "','" "${GPG_KEYS_ENC_ARRAY[@]}")'${CMD_MSG_SIGN:+ & $CMD_MSG_SIGN}"
  run_cmd $(gpg_pass_pipein GPG_PW_SIGN GPG_PW) gpg $CMD_PARAM_SIGN $(gpg_param_passwd GPG_PW_SIGN GPG_PW) $CMD_PARAMS $GPG_USEAGENT --status-fd 1 $GPG_OPTS -o $(qw "${GPG_TEST_PREFIX}_ENC") -e $(qw "$ME_LONG")
  CMD_ERR=$?

  if [ "$CMD_ERR" != "0" ]; then 
    KEY_NOTRUST=$(echo "$CMD_OUT"|awk '/^\[GNUPG:\] INV_RECP 10/ { print $4 }')
    [ -n "$KEY_NOTRUST" ] && HINT="Key '${KEY_NOTRUST}' seems to be untrusted. If you really trust this key try to
  'gpg --edit-key "$KEY_NOTRUST"' and raise the trust level to ultimate. If you
  can trust all of your keys set GPG_OPTS='--trust-model always' in conf file."
    error_gpg_test "Encryption failed (Code $CMD_ERR).${CMD_OUT:+\n$CMD_OUT}" "$HINT"
  fi

  # check decrypting
  CMD_MSG="Test - Decrypt"
  gpg_key_decryptable || CMD_DISABLED="No matching secret key available."
  run_cmd $(gpg_pass_pipein GPG_PW) gpg $(gpg_param_passwd GPG_PW) $GPG_OPTS -o $(qw "${GPG_TEST_PREFIX}_DEC") $GPG_USEAGENT -d $(qw "${GPG_TEST_PREFIX}_ENC")
  CMD_ERR=$?

  if [ "$CMD_ERR" != "0" ]; then 
    error_gpg_test "Decryption failed.${CMD_OUT:+\n$CMD_OUT}"
  fi

# symmetric only
else
  # check encrypting
  CMD_MSG="Test - Encryption with passphrase${CMD_MSG_SIGN:+ & $CMD_MSG_SIGN}"
  run_cmd $(gpg_pass_pipein GPG_PW) gpg $GPG_OPTS $CMD_PARAM_SIGN --passphrase-fd 0 -o $(qw "${GPG_TEST_PREFIX}_ENC") --batch -c $(qw "$ME_LONG")
  CMD_ERR=$?
  if [ "$CMD_ERR" != "0" ]; then 
    error_gpg_test "Encryption failed.${CMD_OUT:+\n$CMD_OUT}"
  fi

  # check decrypting
  CMD_MSG="Test - Decryption with passphrase"
  run_cmd $(gpg_pass_pipein GPG_PW) gpg $GPG_OPTS --passphrase-fd 0 -o $(qw "${GPG_TEST_PREFIX}_DEC") --batch -d $(qw "${GPG_TEST_PREFIX}_ENC")
  CMD_ERR=$?
  if [ "$CMD_ERR" != "0" ]; then 
    error_gpg_test "Decryption failed.${CMD_OUT:+\n$CMD_OUT}"
  fi
fi

# compare original w/ decryptginal
CMD_MSG="Test - Compare"
[ -r "${GPG_TEST_PREFIX}_DEC" ] || CMD_DISABLED="File not found. Nothing to compare."
run_cmd "test \"\$(cat '$ME_LONG')\" = \"\$(cat '${GPG_TEST_PREFIX}_DEC')\""
CMD_ERR=$?
if [ "$CMD_ERR" = "0" ]; then 
  cleanup_gpgtest
else
  error_gpg_test "Comparision failed.${CMD_OUT:+\n$CMD_OUT}"
fi

fi # end disabled

## an empty line
#echo

# Exclude file is needed, create it if necessary
[ -f "$EXCLUDE" ] || touch "$EXCLUDE"

# export only used keys, if bkp not already exists ######################################
gpg_export_if_needed "${GPG_KEYS_ENC_ARRAY[@]}" "$(gpg_signing && echo $GPG_KEY_SIGN)"


# command execution #####################################################################

# urldecode url vars into plain text
var_isset 'TARGET_URL_USER' && TARGET_URL_USER="$(url_decode "$TARGET_URL_USER")"
var_isset 'TARGET_URL_PASS' && TARGET_URL_PASS="$(url_decode "$TARGET_URL_PASS")"

# defined TARGET_USER&PASS vars replace their URL pendants 
# (double defs already dealt with)
var_isset 'TARGET_USER' && TARGET_URL_USER="$TARGET_USER"
var_isset 'TARGET_PASS' && TARGET_URL_PASS="$TARGET_PASS"

TARGET_URL_PROT_lowercase="$(tolower "${TARGET_URL_PROT%%:*}")"

# issue some warnings
case "$TARGET_URL_PROT_lowercase" in
  'cf+http')
    # info on missing AUTH_URL
    if ! var_isset 'CLOUDFILES_AUTHURL'; then
      inform "No CLOUDFILES_AUTHURL exported (in conf).
Will use default which is probably rackspace."
    fi
    ;;
   'swift')
    # info on possibly missing AUTH_URL
       var_isset 'SWIFT_AUTHURL' ||\
       warning "\
Swift will probably fail because the conf var SWIFT_AUTHURL was not exported!"
    ;;
  'rsync')
    # everything in url (this backend does not support pass in env var)
    # this is obsolete from version 0.6.10 (buggy), hopefully fixed in 0.6.11
    # print warning older version is detected
    duplicity_version_lt 610 &&
      warning "\
Duplicity version '$DUPL_VERSION' does not support providing the password as 
env var for rsync backend. For security reasons you should consider to 
update to a version greater than '0.6.10' of duplicity."
    ;;
esac


# for all protocols we put username in url and pass into env var 
# for sec�rity reasons, we url_encode username to protect special chars
# first sortout backends with special ways to handle password
case "$TARGET_URL_PROT_lowercase" in
  'imap'|'imaps')
    var_isset 'TARGET_URL_PASS' && BACKEND_PARAMS="IMAP_PASSWORD=$(qw "${TARGET_URL_PASS}")"
    ;;
  *)
    # rest uses FTP_PASS var
    var_isset 'TARGET_URL_PASS' && \
      BACKEND_PARAMS="FTP_PASSWORD=$(qw "${TARGET_URL_PASS}")"
    ;;
esac
# insert url encoded username into target url if needed
if var_isset 'TARGET_URL_USER' && [ "$TARGET_URL_PROT_lowercase" != "file" ]; then
  BACKEND_URL="${TARGET_URL_PROT}$(url_encode "${TARGET_URL_USER}")@${TARGET_URL_HOSTPATH}"
else
  BACKEND_URL="$TARGET"
fi


# protect eval from special chars in url (e.g. open ')' in password, 
# spaces in path, quotes) happens above in duplify() via quotewrap()
SOURCE="$SOURCE"
BACKEND_URL="$BACKEND_URL"
EXCLUDE="$EXCLUDE"
# since 0.7.03 --exclude-globbing-filelist is deprecated
EXCLUDE_PARAM="--exclude$(duplicity_version_lt 703 && echo -globbing)-filelist" 

# replace magic separators to command equivalents (+=and,-=or,[=groupIn,]=groupOut)
cmds=$(awk -v cmds="$cmds" "BEGIN{ \
  gsub(/\+/,\"_and_\",cmds);\
  gsub(/\-/,\"_or_\",cmds);\
  gsub(/\[/,\"_groupIn_\",cmds);\
  gsub(/\]/,\"_groupOut_\",cmds);\
  print cmds}")

# split commands by '_', preserve spaces even if not allowed :)
IFS='_' read -ra CMDS_IN <<< "$(tolower "$cmds")"

# convert cmds to array, 
# post process, translate batch commands
# ATTENTION: commands are lowercase from here on out
declare -a CMDS
for cmd in "${CMDS_IN[@]}"; do
  case "$cmd" in
    # backup -> [pre_bkp_post]
    'backup')
      CMDS=("${CMDS[@]}" groupin pre bkp post groupout)
    ;;
    # purgeAuto -> [purge purgeFull purgeIncr] depending on set conf vars 
    'purgeauto')
      purgeAuto=${MAX_AGE:+ purge}${MAX_FULL_BACKUPS:+ purgefull}${MAX_FULLS_WITH_INCRS:+ purgeincr}
      [[ -z "$purgeAuto" ]] && error "Command 'fullAuto' was given but neither of the purge conf vars configured."
      CMDS=("${CMDS[@]}" groupin $purgeAuto groupout)
    ;;
    *)
      CMDS=("${CMDS[@]}" "$cmd")
    ;;
  esac
done
#echo $(IFS=',';echo "${CMDS[*]}")

unset FTPL_ERR

# run CMDS
for cmd in "${CMDS[@]}";
do

## init
# raise index in cmd array for pre/post param
var_isset 'CMD_NO' && CMD_NO=$((++CMD_NO)) || CMD_NO=0

unset CMD_VALUE CMD_NEXT CMD_PREV CND_NEXT CND_PREV

# get next cmd,cnd vars
nextno=$(( $CMD_NO ))
while ! var_isset 'CMD_NEXT'
do
  nextno=$(($nextno+1))
  if [ "$nextno" -lt "${#CMDS[@]}" ]; then
    CMD_VALUE=${CMDS[$nextno]}
    is_condition "$CMD_VALUE" && CND_NEXT="$CMD_VALUE" && continue
    is_groupMarker "$CMD_VALUE" && continue
    CMD_NEXT="$CMD_VALUE"
  else
    CMD_NEXT='END'
  fi
done

# get prev cnd, cnds are skipped pseudocmds
prevno=$(( $CMD_NO ));
while ! var_isset 'CND_PREV'
do
  prevno=$(($prevno-1))
  if [ "$prevno" -ge 0 ]; then
    CMD_VALUE=${CMDS[$prevno]}
    is_condition "$CMD_VALUE" && CND_PREV="$CMD_VALUE" && break
    is_command "$CMD_VALUE" && break
  else
    break
  fi
done

# get prev cmd command minus skipped commands, only executed
prevno=$(( $CMD_NO - ${CMD_SKIPPED-0} ));
while ! var_isset 'CMD_PREV'
do
  prevno=$(($prevno-1))
  if [ "$prevno" -ge 0 ]; then
    CMD_VALUE=${CMDS[$prevno]}
    is_condition "$CMD_VALUE" && CND_PREV="$CMD_VALUE" && continue
    is_groupMarker "$CMD_VALUE" && continue
    CMD_PREV="$CMD_VALUE"
  else
    CMD_PREV='START'
  fi
done

function get_cmd_skip_count {
  # find closing bracket, get group skip count
  local nextno=$CMD_NO
  local GRP_OPEN=0
  local GRP_SKIP=0
  local CMD_VALUE
  while [ "$nextno" -lt "${#CMDS[@]}" ]
  do
    nextno=$(($nextno+1))
    CMD_VALUE=${CMDS[$nextno]}
    GRP_SKIP=$(( ${GRP_SKIP} + 1 ));
    if is_command "$CMD_VALUE" && [ "$GRP_OPEN" -lt 1 ]; then
        break;
    elif [ "$CMD_VALUE" == 'groupin' ]; then
      GRP_OPEN=$(( ${GRP_OPEN} + 1 ))
    elif [ "$CMD_VALUE" == 'groupout' ]; then
      GRP_OPEN=$(( ${GRP_OPEN} - 1 ))
      if [ "$GRP_OPEN" -lt 1 ]; then
        break;
      fi
    fi
  done

  echo $GRP_SKIP;
}

# decision time: are we skipping already or dealing with condition "commands" or other non-cmds?
unset SKIP_NOW
if var_isset 'CMD_SKIP' && [ $CMD_SKIP -gt 0 ]; then
  # skip cnd/grp cmds silently
  is_command "$cmd" && echo -e "\n--- Skipping command $(toupper $cmd) ! ---"
  CMD_SKIP=$(($CMD_SKIP - 1))
  SKIP_NOW="yes"
elif ! var_isset 'PREVIEW' && [ "$cmd" == 'and' ] && [ "$CMD_ERR" -ne "0" ]; then
  CMD_SKIP=$(get_cmd_skip_count)
  # incl. this "cmd"
  CMD_SKIP=$(( $CMD_SKIP + 1 ))
  unset CMD_SKIPPED
  SKIP_NOW="yes"
elif ! var_isset 'PREVIEW' && [ "$cmd" == 'or' ] && [ "$CMD_ERR" -eq "0" ]; then
  CMD_SKIP=$(get_cmd_skip_count)
  # incl. this "cmd"
  CMD_SKIP=$(( $CMD_SKIP + 1 ))
  unset CMD_SKIPPED
  SKIP_NOW="yes"
elif is_condition "$cmd" || is_groupMarker "$cmd"; then
  unset 'CMD_SKIP';
  SKIP_NOW="yes"
fi

# let's do the skip now
if [ -n "$SKIP_NOW" ]; then
  # sum up how many commands we actually skipped for the prev var routines
  CMD_SKIPPED=$((${CMD_SKIPPED-0} + 1))
  continue
fi

# save start time
RUN_START=$(nsecs)

# export some useful env vars for external scripts/programs to use
export PROFILE CONFDIR SOURCE TARGET_URL_PROT TARGET_URL_HOSTPATH \
       TARGET_URL_USER TARGET_URL_PASS \
       GPG_KEYS_ENC=$(join "\n" "${GPG_KEYS_ENC_ARRAY[@]}") GPG_KEY_SIGN \
       GPG_PW CMD_PREV CMD_NEXT CMD_ERR CND_PREV CND_NEXT\
       RUN_START

# user info
echo; separator "Start running command $(toupper $cmd) at $(datefull_from_nsecs $RUN_START)"

case "$(tolower $cmd)" in
  'pre'|'post')
    if [ "$cmd" == 'pre' ]; then
      script=$PRE
    else
      script=$POST
    fi
    # script execution in a subshell, protect us from failures/var overwrites
    ( run_script "$script" )
    ;;
  'bkp')
    duplify backup -- "${dupl_opts[@]}" $EXCLUDE_PARAM "$EXCLUDE" \
          "$SOURCE" "$BACKEND_URL"
    ;;
  'incr')
    duplify incremental -- "${dupl_opts[@]}" $EXCLUDE_PARAM "$EXCLUDE" \
          "$SOURCE" "$BACKEND_URL"
    ;;
  'full')
    duplify full -- "${dupl_opts[@]}" $EXCLUDE_PARAM "$EXCLUDE" \
          "$SOURCE" "$BACKEND_URL"
    ;;
  'verify')
    TIME="${ftpl_pars[0]:+"-t ${ftpl_pars[0]}"}"
    duplify verify -- $TIME "${dupl_opts[@]}" $EXCLUDE_PARAM "$EXCLUDE" \
          "$BACKEND_URL" "$SOURCE"
    ;;
  'verifypath')
    TIME="${ftpl_pars[2]:+"-t ${ftpl_pars[2]}"}"
    IN_PATH="${ftpl_pars[0]}"; OUT_PATH="${ftpl_pars[1]}"; 
    ( [ -z "$IN_PATH" ] || [ -z "$OUT_PATH" ] ) && error "  Missing parameter <rel_bkp_path> or <local_path> for verifyPath.
  
  Hint: 
    Syntax is -> $ME <profile> verifyPath <rel_bkp_path> <local_path> [<age>]"

    duplify verify -- $TIME "${dupl_opts[@]}" $EXCLUDE_PARAM "$EXCLUDE" \
          --path-to-restore "$IN_PATH" "$BACKEND_URL" "$OUT_PATH"
    ;;
  'list')
    # time param exists since 0.5.10+
    TIME="${ftpl_pars[0]:+"-t ${ftpl_pars[0]}"}"
    duplify list-current-files -- $TIME "${dupl_opts[@]}" "$BACKEND_URL"
    ;;
  'cleanup')
    duplify cleanup -- "${dupl_opts[@]}" "$BACKEND_URL"
    ;;
  'purge')
    MAX_AGE=${ftpl_pars[0]:-$MAX_AGE}
    [ -z "$MAX_AGE" ] && error "  Missing parameter <max_age>. Can be set in profile or as command line parameter."
    
    duplify remove-older-than "${MAX_AGE}" \
          -- "${dupl_opts[@]}" "$BACKEND_URL"
    ;;
  'purgefull')
    MAX_FULL_BACKUPS=${ftpl_pars[0]:-$MAX_FULL_BACKUPS}
    [ -z "$MAX_FULL_BACKUPS" ] && error "  Missing parameter <max_full_backups>. Can be set in profile or as command line parameter."
  
    duplify remove-all-but-n-full "${MAX_FULL_BACKUPS}" \
          -- "${dupl_opts[@]}" "$BACKEND_URL"
    ;;
  'purgeincr')
    MAX_FULLS_WITH_INCRS=${ftpl_pars[0]:-$MAX_FULLS_WITH_INCRS}
    [ -z "$MAX_FULLS_WITH_INCRS" ] && error "  Missing parameter <max_fulls_with_incrs>. Can be set in profile or as command line parameter."
  
    duplify remove-all-inc-of-but-n-full "${MAX_FULLS_WITH_INCRS}" \
          -- "${dupl_opts[@]}" "$BACKEND_URL"
    ;;
  'restore')
    OUT_PATH="${ftpl_pars[0]}"; TIME="${ftpl_pars[1]:-now}";
    [ -z "$OUT_PATH" ] && error "  Missing parameter target_path for restore.
  
  Hint: 
    Syntax is -> $ME <profile> restore <target_path> [<age>]"

    duplify restore -- -t "$TIME" "${dupl_opts[@]}" "$BACKEND_URL" "$OUT_PATH"
    ;;
  'fetch')
    IN_PATH="${ftpl_pars[0]}"; OUT_PATH="${ftpl_pars[1]}"; 
    TIME="${ftpl_pars[2]:-now}";
    ( [ -z "$IN_PATH" ] || [ -z "$OUT_PATH" ] ) && error "  Missing parameter <src_path> or <target_path> for fetch.

  Hint: 
    Syntax is -> $ME <profile> fetch <src_path> <target_path> [<age>]"

    duplify restore -- --restore-time "$TIME" "${dupl_opts[@]}" \
              --path-to-restore "$IN_PATH" "$BACKEND_URL" "$OUT_PATH"
    ;;
  'status')
    duplify collection-status -- "${dupl_opts[@]}" "$BACKEND_URL"
    ;;    
  *)
    error "  Unknown command '$cmd'.

  Hint:
    Use '$ME usage' to get usage help."
    ;;
esac

CMD_ERR=$?
RUN_END=$(nsecs)
RUNTIME=$(( $RUN_END - $RUN_START ))

# print message on error; set error code
if [ "$CMD_ERR" -ne 0 ]; then
  error_print "$(datefull_from_nsecs $RUN_END) Task '$(echo $cmd|awk '$0=toupper($0)')' failed with exit code '$CMD_ERR'."
  FTPL_ERR=1
fi

separator "Finished state $(error_to_string $CMD_ERR) at $(datefull_from_nsecs $RUN_END) - \
Runtime $(printf "%02d:%02d:%02d.%03d" $((RUNTIME/1000000000/60/60)) $((RUNTIME/1000000000/60%60)) $((RUNTIME/1000000000%60)) $((RUNTIME/1000000%1000)) )"

done

exit ${FTPL_ERR}
