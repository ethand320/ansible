#!/usr/bin/env bash
#
# Joseph Wegner
# 2015-07-21
# migrate.sh
#
# Script to migrate client from RHN Classic to Satellite 6
# fsanche8's directions: https://knowledge.depaul.edu/pages/viewpage.action?pageId=77793148

USAGE="USAGE: $0 ARGS

Required Arguments (specify one per group below):
  Lifecycle Environment
    -l|--lib      Join client to the library lifecycle
    -t|--test     Join client to the testing lifecycle
    -p|--prod     Join client to the production lifecycle
  
  IS Group
    -o|--oss	Specify client as OSS managed
    -b|--bcs	Specify client as BCS managed
    -n|--nti	Specify client as NTI managed

Optional Arguments:
  -i|--i386	client is 32-bit (only for OSS RHEL 5 repos)
  -a|--auto	run the script in automatic mode (non-interactive)
  -h|--help     print this usage message and exit"

# Check that we have root
if [[ "$(whoami)" != "root" ]] ; then echo -e "Insufficient permissions. $0 must be run as root." >&2 ; exit 1 ; fi

# If no args passed, print usage
if [[ -z "$1" ]]; then echo -e "$USAGE"; exit 0 ; fi

# Set initial values for parameters
LIB=""
TEST=""
PROD=""
OSS=""
BCS=""
NTI=""
i386=""
AUTO=""

# Parse arguments
TEMP=`getopt -o ltpobniah -l lib,test,prod,oss,bcs,nti,i386,auto,help -n "$0" -- "$@"`

# Bad parameters
if [[ "$?" -ne 0 ]] ; then echo -e "$USAGE" ; echo -e "Terminating..." >&2 ; exit 1 ; fi

eval set -- "$TEMP"

while true ; do
  case "$1" in
    -l|--lib)	LIB="T" ; shift ;;
    -t|--test)	TEST="T" ; shift ;;
    -p|--prod)	PROD="T" ; shift ;;
    -o|--oss)	OSS="T" ; shift ;;
    -b|--bcs)	BCS="T" ; shift ;;
    -n|--nti)	NTI="T" ; shift ;;
    -i|--i386)	i386="T" ; shift ;;
    -a|--auto)	AUTO="T" ; shift ;;
    -h|--help)	echo -e "$USAGE" ; exit 0 ;;
    --)		shift ; break ;;
    *)		echo -e "Internal error." >&2 ; exit 1 ;;
  esac
done

# Make sure that only one option is entered for environment type
if [[ "$LIB" = "T" && "$TEST" = "T" ]] || [[ "$LIB" = "T" && "$PROD" = "T" ]] || [[ "$TEST" = "T" && "$PROD" = "T" ]] ; then
  echo -e "Client cannot have more than one lifecycle environment." >&2
  exit 1
fi

# Make sure that only one option is entered for management group type
if [[ "$OSS" = "T" && "$BCS" = "T" ]] || [[ "$OSS" = "T" && "$NTI" = "T" ]] || [[ "$BCS" = "T" && "$NTI" = "T" ]] ; then
  echo -e "Client cannot be managed by more than one IS group." >&2
  exit 1
fi

# Determine what RHEL version we're running
VERSION_LONG=`cat /etc/redhat-release | awk {'print $7'}`
VERSION="${VERSION_LONG:0:1}"

# Ensure 32-bit only enabled for OSS RHEL 5
if [[ "$i386" = "T" ]] && [[ "$OSS" != "T"  || "$VERSION" != "5" ]] ; then
  echo -e "32-bit clients are only supported for OSS RHEL 5 repos." >&2
  exit 1
fi

# Store lifecycle environment into variable (simplifies case later)
ENV=""
if [[ "$LIB" = "T" ]] ; then ENV="LIB" ; fi
if [[ "$TEST" = "T" ]] ; then ENV="TEST" ; fi
if [[ "$PROD" = "T" ]] ; then ENV="PROD" ; fi
if [[ "$ENV" = "" ]] ; then echo -e "You must select a lifecycle environment." >&2 ; exit 1 ; fi

# Store lifecycle environment into variable (simplifies case later)
GROUP=""
if [[ "$OSS" = "T" ]] ; then GROUP="OSS" ; fi
if [[ "$BCS" = "T" ]] ; then GROUP="BCS" ; fi
if [[ "$NTI" = "T" ]] ; then GROUP="NTI" ; fi
if [[ "$GROUP" = "" ]] ; then echo -e "You must select an IS group." >&2 ; exit 1 ; fi

# Satellite subscription keys for each release
KEY_5_i386_OSS="rhel5satkeyi386cc6d8fc76beba8b8487aaf25a8ab00e6"
KEY_5_OSS="rhel5satkey666a865675f0cebc5b5b53d6c7170d2c"
KEY_5_BCS="rhel5satkeybcs0ea154ecb599447ccafb9bb84e6c9c74"
KEY_5_NTI="rhel5satkeyntia126a01efb594a8009ab363f7acb4bf4"
KEY_6_OSS="rhel6satkey6b6d5dd8a676e369a4e9aed224f7f23d"
KEY_6_BCS="rhel6satkeybcsa3665f2fe47741a76a757a4fd5dd9ac1"
KEY_6_NTI="rhel6satkeynti269965cf85dec20a2b849cbb66953942"
KEY_7_OSS="rhel7satkey281bc9355c731499c1b1b224de108fcc3"
KEY_7_BCS="rhel7satkeybcsc709e56e90e6dc3efc0d9c7405236c53"
KEY_7_NTI="rhel7satkeyntia95e1fe6d45e96cb15a808d620d87399"

# OSS Puppet environments
ENV_5_i386_OSS_LIB="KT_depaul_university_Library_RHEL_5_i386_15"
ENV_5_i386_OSS_TEST="KT_depaul_university_Testing_RHEL_5_i386_15"
ENV_5_i386_OSS_PROD="KT_depaul_university_Production_RHEL_5_i386_15"
ENV_5_OSS_LIB="KT_depaul_university_Library_RHEL_5_X86_64_9"
ENV_5_OSS_TEST="KT_depaul_university_Testing_RHEL_5_X86_64_9"
ENV_5_OSS_PROD="KT_depaul_university_Production_RHEL_5_X86_64_9"
ENV_6_OSS_LIB="KT_depaul_university_Library_RHEL_6_x86_64_7"
ENV_6_OSS_TEST="KT_depaul_university_Testing_RHEL_6_x86_64_7"
ENV_6_OSS_PROD="KT_depaul_university_Production_RHEL_6_x86_64_7"
ENV_7_OSS_LIB="KT_depaul_university_Library_RHEL_7_x86_64_6"
ENV_7_OSS_TEST="KT_depaul_university_Testing_RHEL_7_x86_64_6"
ENV_7_OSS_PROD="KT_depaul_university_Production_RHEL_7_x86_64_6"

# BCS Puppet Environments
ENV_5_BCS_LIB="KT_depaul_university_Library_RHEL_5_x86_64_BCS_11"
ENV_5_BCS_TEST="KT_depaul_university_Testing_RHEL_5_x86_64_BCS_11"
ENV_5_BCS_PROD="KT_depaul_university_Production_RHEL_5_x86_64_BCS_11"
ENV_6_BCS_LIB="KT_depaul_university_Library_RHEL_6_x86_64_BCS_8"
ENV_6_BCS_TEST="KT_depaul_university_Testing_RHEL_6_x86_64_BCS_8"
ENV_6_BCS_PROD="KT_depaul_university_Production_RHEL_6_x86_64_BCS_8"
ENV_7_BCS_LIB="KT_depaul_university_Library_RHEL_6_x86_64_BCS_8"
ENV_7_BCS_TEST="KT_depaul_university_Testing_RHEL_7_x86_64_BCS_14"
ENV_7_BCS_PROD="KT_depaul_university_Production_RHEL_7_x86_64_BCS_14"

# NTI Puppet Environments
ENV_5_NTI_LIB="KT_depaul_university_Library_RHEL_5_x86_64_NTI_12"
ENV_5_NTI_TEST="KT_depaul_university_Testing_RHEL_5_x86_64_NTI_12"
ENV_5_NTI_PROD="KT_depaul_university_Production_RHEL_5_x86_64_NTI_12"
ENV_6_NTI_LIB="KT_depaul_university_Library_RHEL_6_x86_64_NTI_10"
ENV_6_NTI_TEST="KT_depaul_university_Testing_RHEL_6_x86_64_NTI_10"
ENV_6_NTI_PROD="KT_depaul_university_Production_RHEL_6_x86_64_NTI_10"
ENV_7_NTI_LIB="KT_depaul_university_Library_RHEL_7_x86_64_NTI_13"
ENV_7_NTI_TEST="KT_depaul_university_Testing_RHEL_7_x86_64_NTI_13"
ENV_7_NTI_PROD="KT_depaul_university_Production_RHEL_7_x86_64_NTI_13"

# determine directory paths
WORKING_DIR=`pwd`
if [[ "$i386" = "T" ]] ; then
  PACKAGE_NAME="sat-rhel$VERSION-i386"
else
  PACKAGE_NAME="sat-rhel$VERSION"
fi
PACKAGE_DIR="$WORKING_DIR/$PACKAGE_NAME"

# Remove Spacewalk packages if installed
echo -e "Removing Spacewalk packages if they're installed..."
SPACEWALK=`rpm -qa | grep spacewalk`
if [[ "$SPACEWALK" != "" ]] ; then
  if [[ "$AUTO" = "T" ]] ; then 
    yum -y erase spacewalk-client-repo spacewalk-backend-libs
  else
    yum erase spacewalk-client-repo spacewalk-backend-libs
  fi
fi

# check if package has been downloaded already
if [[ ! -e "$WORKING_DIR/$PACKAGE_NAME.tar.gz" ]] ; then
  # Download and install packages for this version from Satellite's public webserver
  wget --no-check-certificate "https://satelliteprd01.is.depaul.edu/pub/$PACKAGE_NAME.tar.gz"
  tar xf "$WORKING_DIR/$PACKAGE_NAME.tar.gz"
fi
PACKAGES=""
for i in `ls "$PACKAGE_DIR" | grep rpm` ; do PACKAGES+=" $PACKAGE_DIR/$i" ; done
if [[ "$AUTO" = "T" ]] ; then
  eval "yum --nogpgcheck -y install $PACKAGES"
else
  eval "yum --nogpgcheck install $PACKAGES"
fi

if [[ "$AUTO" = "T" ]] ; then
  # Change RHN config files to point client to Satellite
  sed -i.old 's/^\(serverURL=\).*/\1https:\/\/satelliteprd01.is.depaul.edu\/XMLRPC/' /etc/sysconfig/rhn/up2date
  sed -i.old 's/^\(enabled = \).*/\10/' /etc/yum/pluginconf.d/rhnplugin.conf
else
  VERIFY=""
  read -p "Overwrite current RHN configuration? (y/n) " VERIFY
  if [[ "$VERIFY" = "y" ]] ; then
    # Change RHN config files to point client to Satellite
    sed -i.old 's/^\(serverURL=\).*/\1https:\/\/satelliteprd01.is.depaul.edu\/XMLRPC/' /etc/sysconfig/rhn/up2date
    sed -i.old 's/^\(enabled = \).*/\10/' /etc/yum/pluginconf.d/rhnplugin.conf
  fi
fi

# Put cert and pem in correct folder, keeping old file just in case
if [[ "$AUTO" = "T" ]] ; then
  if [ -a /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT ] ; then
    mv /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT.old
  fi
  mv "$PACKAGE_DIR/RHN-ORG-TRUSTED-SSL-CERT" /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT
else
  VERIFY=""
  read -p "Add new RHN SSL certificate? (y/n) " VERIFY
  if [[ "$VERIFY" = "y" ]] ; then
    if [ -a /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT ] ; then
      mv /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT.old
    fi
    mv "$PACKAGE_DIR/RHN-ORG-TRUSTED-SSL-CERT" /usr/share/rhn/RHN-ORG-TRUSTED-SSL-CERT
  fi
fi

if [[ "$AUTO" = "T" ]] ; then
  if [ -a /etc/pki/product/69.pem ] ; then
    mv /etc/pki/product/69.pem /etc/pki/product/69.pem.old
  fi
  mv "$PACKAGE_DIR/69.pem" /etc/pki/product/69.pem
else
  VERIFY=""
  read -p "Add new RHN .pem file? (y/n) " VERIFY
  if [[ "$VERIFY" = "y" ]] ; then
    if [ -a /etc/pki/product/69.pem ] ; then
      mv /etc/pki/product/69.pem /etc/pki/product/69.pem.old
    fi
    mv "$PACKAGE_DIR/69.pem" /etc/pki/product/69.pem
  fi
fi

# Choose key based on release version
KEY=""
PUP_ENV=""

case "$VERSION" in 
  5)	case "$GROUP" in 	
	  "OSS")
		if [[ "$i386" = "T" ]] ; then
		  KEY=$KEY_5_i386_OSS 
		  case "$ENV" in
		    "LIB")	PUP_ENV="$ENV_5_i386_OSS_LIB" ;;
		    "TEST")	PUP_ENV="$ENV_5_i386_OSS_TEST" ;;
		    "PROD")	PUP_ENV="$ENV_5_i386_OSS_PROD" ;;
		    *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
		  esac
		else
		  KEY=$KEY_5_OSS 
		  case "$ENV" in
		    "LIB")	PUP_ENV="$ENV_5_OSS_LIB" ;;
		    "TEST")	PUP_ENV="$ENV_5_OSS_TEST" ;;
		    "PROD")	PUP_ENV="$ENV_5_OSS_PROD" ;;
		    *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
		  esac
		fi
		;;
	  "BCS")
                KEY=$KEY_5_BCS
                case "$ENV" in
                  "LIB")        PUP_ENV="$ENV_5_BCS_LIB" ;;
                  "TEST")       PUP_ENV="$ENV_5_BCS_TEST" ;;
                  "PROD")       PUP_ENV="$ENV_5_BCS_PROD" ;;
                  *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
                esac
                ;;
	  "NTI")
                KEY=$KEY_5_NTI
                case "$ENV" in
                  "LIB")        PUP_ENV="$ENV_5_NTI_LIB" ;;
                  "TEST")       PUP_ENV="$ENV_5_NTI_TEST" ;;
                  "PROD")       PUP_ENV="$ENV_5_NTI_PROD" ;;
                  *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
                esac
                ;;
	  *)	echo -e "Invalid IS group $GROUP" >&2 ; exit 1 ;;
	esac
	;;
  6)	case "$GROUP" in
	  "OSS")
		KEY=$KEY_6_OSS
		case "$ENV" in
		  "LIB")	PUP_ENV="$ENV_6_OSS_LIB" ;;
		  "TEST")	PUP_ENV="$ENV_6_OSS_TEST" ;;
		  "PROD")	PUP_ENV="$ENV_6_OSS_PROD" ;;
		  *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
		esac
		;;
          "BCS")
                KEY=$KEY_6_BCS
                case "$ENV" in
                  "LIB")        PUP_ENV="$ENV_6_BCS_LIB" ;;
                  "TEST")       PUP_ENV="$ENV_6_BCS_TEST" ;;
                  "PROD")       PUP_ENV="$ENV_6_BCS_PROD" ;;
                  *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
                esac
                ;;
          "NTI")
                KEY=$KEY_6_NTI
                case "$ENV" in
                  "LIB")        PUP_ENV="$ENV_6_NTI_LIB" ;;
                  "TEST")       PUP_ENV="$ENV_6_NTI_TEST" ;;
                  "PROD")       PUP_ENV="$ENV_6_NTI_PROD" ;;
                  *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
                esac
                ;;
          *)    echo -e "Invalid IS group $GROUP" >&2 ; exit 1 ;;
        esac
        ;;
  7)	case "$GROUP" in
	  "OSS")
		KEY=$KEY_7_OSS
		case "$ENV" in
		  "LIB")	PUP_ENV="$ENV_7_OSS_LIB" ;;
		  "TEST")	PUP_ENV="$ENV_7_OSS_TEST" ;;
		  "PROD")	PUP_ENV="$ENV_7_OSS_PROD" ;;
		  *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
		esac
		;;
          "BCS")
                KEY=$KEY_7_BCS
                case "$ENV" in
                  "LIB")        PUP_ENV="$ENV_7_BCS_LIB" ;;
                  "TEST")       PUP_ENV="$ENV_7_BCS_TEST" ;;
                  "PROD")       PUP_ENV="$ENV_7_BCS_PROD" ;;
                  *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
                esac
                ;;
          "NTI")
                KEY=$KEY_7_NTI
                case "$ENV" in
                  "LIB")        PUP_ENV="$ENV_7_NTI_LIB" ;;
                  "TEST")       PUP_ENV="$ENV_7_NTI_TEST" ;;
                  "PROD")       PUP_ENV="$ENV_7_NTI_PROD" ;;
                  *) echo -e "Invalid environment $ENV" >&2 ; exit 1 ;;
                esac
                ;;
          *)    echo -e "Invalid IS group $GROUP" >&2 ; exit 1 ;;
        esac
        ;;
  *) echo -e "Invalid release version $VERSION" >&2 ; exit 1 ;;
esac

# Clean subscription manager's local data
if [[ "$AUTO" = "T" ]] ; then 
  echo -e "Clearing Subscription Manager's local data..."
  yum clean all
  subscription-manager clean
else
  VERIFY=""
  read -p "Do you want to clean subscription manager's local data? (y/n) " VERIFY
  if [ "$VERIFY" = "y" ] ; then
    echo -e "Clearing Subscription Manager's local data..."
    yum clean all
    subscription-manager clean
  fi
fi

# Register with subscription manager
if [[ "$AUTO" = "T" ]] ; then 
  echo -e "Registering with Satellite server using key $KEY"
  eval "subscription-manager register --org depaul_university --activationkey $KEY"
else
  VERIFY=""
  read -p "Do you want to register with the Satellite server using key $KEY? (y/n) " VERIFY
  if [ "$VERIFY" = "y" ] ; then
    echo -e "Registering with Satellite server using key $KEY"
    eval "subscription-manager register --org depaul_university --activationkey $KEY"
  fi
fi

if [[ "$AUTO" = "" ]] ; then 
  subscription-manager repos
  VERIFY=""
  read -p "Client should have auto-attached. Did our custom repos show up? (y/n) " VERIFY
  if [ "$VERIFY" != "y" ] ; then
    echo -e "You will need to configure the rest by hand. See https://knowledge.depaul.edu/pages/viewpage.action?pageId=77793148 for details."
    exit 1
  fi
fi

# Install katello and puppet
echo -e "Installing Puppet and Katello..."
if [[ "$AUTO" = "T" ]] ; then 
  yum -y install katello-agent puppet
else
  yum install katello-agent puppet
fi

if [[ "$AUTO" = "T" ]] ; then 
  # Lines to append to puppet config depending on environment
  PUP_CONF="\    pluginsync = true\n    report = true\n    ignoreschedules = true\n    daemon = false\n    ca_server = satelliteprd01.is.depaul.edu\n    server = satelliteprd01.is.depaul.edu\n    environment = $PUP_ENV"
  # add lines to puppet config
  sed -i.old "/.*classfile.*/a ${PUP_CONF}" /etc/puppet/puppet.conf
else
  VERIFY=""
  read -p "Do you want to modify the Puppet configuration? (y/n) " VERIFY
  if [ "$VERIFY" = "y" ] ; then
    # Lines to append to puppet config depending on environment
    PUP_CONF="\    pluginsync = true\n    report = true\n    ignoreschedules = true\n    daemon = false\n    ca_server = satelliteprd01.is.depaul.edu\n    server = satelliteprd01.is.depaul.edu\n    environment = $PUP_ENV"
    # add lines to puppet config
    sed -i.old "/.*classfile.*/a ${PUP_CONF}" /etc/puppet/puppet.conf
  fi
fi

# Request certificate from satellite server
if [[ "$AUTO" = "T" ]] ; then 
  echo -e "Requesting certificate from the Satelite server..."
  puppet agent -t --server satelliteprd01.is.depaul.edu
else
  VERIFY=""
  read -p "Do you want to request a certificate from the Satellite server? (y/n) " VERIFY
  if [[ "$VERIFY" = "y" ]] ; then
    echo -e "Requesting certificate from the Satelite server..."
    puppet agent -t --server satelliteprd01.is.depaul.edu
  fi
fi

# Enable puppet daemon to start at boot
if [[ "$AUTO" = "T" ]] ; then 
  if [ "$VERSION" == "7" ]; then
    systemctl enable puppet
  else
    chkconfig puppet on
  fi
else
  VERIFY=""
  read -p "Do you want the Puppet daemon to start at boot? (y/n) " VERIFY
  if [ "$VERIFY" = "y" ] ; then
    if [ "$VERSION" == "7" ]; then
      systemctl enable puppet
    else
      chkconfig puppet on
    fi
  fi
fi

# 2015-07-24 Frank believes starting the Puppet service before signing
# the certificate may be causing errors, so disabling for now
# # start the puppet daemon
# if [ "$VERSION" == "7" ]; then
#   systemctl restart puppet
# else
#   service puppet restart
# fi

echo -e "The client should have requested a certificate from the
Satellite server. You will need to enter the administrative console to
sign the certificate. 
After the certificate is signed, you will need to run

  # puppet agent -tv --server satelliteprd01.is.depaul.edu

to add the client to the puppet environment. 

After the certificate is signed, you should then start the Puppet service
using the appropriate command for the client.

Finally, you must add the client to the proper organization and location from
the Satellite administrative console"
