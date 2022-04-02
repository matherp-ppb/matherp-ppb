# Welcome
Hellow World!

## Mac Book Rebuild 2022
Port from https://confluence.app.betfair/display/~MatherP/Mac+book+Rebuild+2022

## Setup Docker Desktop & use for dev envs
We have a license for Docker Desktop so can install as per https://medium.com/containers-101/local-kubernetes-for-mac-minikube-vs-docker-desktop-f2789b3cad3a

If you ask in #pam-macos-support they can enable you to use...

```sudo /Applications/Docker.app/Contents/MacOS/Docker --install-privileged-components```

So download the intel docker from https://docs.docker.com/desktop/mac/install/, move to applications and then run the above

I enabled Start Docker Desktop when you log in,  Enable Kubernetes, Use Docker Compose v2, Always download updates, Use the new Virtualization framework and Enable VirtioFS accelerated directory sharing. It states installing kubernetes and the restarts the program.

## Adventures in CDK

We're getting to steal from https://medium.com/dataengineerbr/creating-a-local-environment-to-develop-on-aws-cdk-with-docker-and-vscode-f26569d30870 to build containers for CDK developement so I copied the contents to https://github.com/matherp-ppb/docker-dev-envs and tweaked it a bit.

```
sudo docker build -t matherpppb/alpine3.13-aws2-cdk1.91.0 .
docker run -itd -v ~/.aws/:/root/.aws -v /Users/matherp/:/home/matherp --name alpine-aws2-cdk matherpppb/alpine3.13-aws2-cdk1.91.0 bash
docker exec -it alpine-aws2-cdk /bin/sh

aws sso login --no-browser
Browser will not be automatically opened.
Please visit the following URL:

https://device.sso.eu-west-1.amazonaws.com/

Then enter the code:

XXXX-XXXX

Alternatively, you may visit the following URL which will autofill the code upon loading:
https://device.sso.eu-west-1.amazonaws.com/?user_code=XXXX-XXXX
Successfully logged into Start URL: https://d-93670ced1d.awsapps.com/start
/opt/app # aws sts get-caller-identity
> aws sts get-caller-identity
{
    "UserId": "AROA5GJT5SJ7XPAABPOE6:philip.mather@paddypowerbetfair.com",
    "Account": "906883863167",
    "Arn": "arn:aws:sts::906883863167:assumed-role/AWSReservedSSO_InfraAdmin_ddf68f2f2ef15701/philip.mather@paddypowerbetfair.com"
}
/opt/app #
```


# Install latest AWS CLI V2
This was to my mac proper rather than to a container
```
22:03:01 matherp@MACC02X9DETJG5H in ~ on 🅰 (eu-west-1) ➜ curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
% Total % Received % Xferd Average Speed Time Time Time Current
Dload Upload Total Spent Left Speed
100 22.5M 100 22.5M 0 0 4369k 0 0:00:05 0:00:05 --:--:-- 5877k
22:12:24 matherp@MACC02X9DETJG5H in ~ on 🅰 (eu-west-1) ➜ sudo installer -pkg AWSCLIV2.pkg -target /
Password:
installer: Package name is AWS Command Line Interface
installer: Installing at base path /
installer: The install was successful.
 
22:12:48 matherp@MACC02X9DETJG5H in ~ on 🅰 (eu-west-1) ➜ aws --version
aws-cli/2.1.24 Python/3.7.4 Darwin/19.6.0 exe/x86_64 prompt/off
```

Configure AWS SSO CLI access
Reference is here... https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html
Slightly annoying thing is doco'ed here... https://stackoverflow.com/questions/58844640/aws2-sso-configuration-error-after-installing-aws2-cli-on-macos
```
10:54:24 matherp@MACC02X9DETJG5H in ~ on 🅰 (eu-west-1) ➜ aws configure sso
SSO start URL [None]: https://d-93670ced1d.awsapps.com/
SSO Region [None]: eu-west-1
 
An error occurred (InvalidRequestException) when calling the StartDeviceAuthorization operation:
10:54:43 matherp@MACC02X9DETJG5H in ~ on 🅰 (eu-west-1) ✗ rm -rf .aws/sso/
10:55:00 matherp@MACC02X9DETJG5H in ~ on 🅰 (eu-west-1) ➜ aws configure sso
SSO start URL [None]: https://d-93670ced1d.awsapps.com/start
SSO Region [None]: eu-west-1
Attempting to automatically open the SSO authorization page in your default browser.
If the browser does not open or you wish to use a different device to authorize this request, open the following URL:
 
https://device.sso.eu-west-1.amazonaws.com/
 
Then enter the code:
 
XXXX-XXXX
There are 3 AWS accounts available to you.
Using the account ID 906883863167
The only role available to you is: InfraAdmin
Using the role name "InfraAdmin"
CLI default client Region [eu-west-1]:
CLI default output format [None]: json
CLI profile name [InfraAdmin-906883863167]: aws-lz-dev-infraAdmin
 
To use this profile, specify the profile name using --profile, as shown:
 
aws s3 ls --profile aws-lz-dev-infraAdmin
 
11:00:57 matherp@MACC02X9DETJG5H in ~ on 🅰 (eu-west-1) ✗ aws sts get-caller-identity --profile aws-lz-dev-infraAdmin
{
"UserId": "AROA5GJT5SJ7XPAABPOE6:philip.mather@paddypowerbetfair.com",
"Account": "906883863167",
"Arn": "arn:aws:sts::906883863167:assumed-role/AWSReservedSSO_InfraAdmin_ddf68f2f2ef15701/philip.mather@paddypowerbetfair.com"
}
```
This process adds the following stanza to the bottom of .aws/config...
```
[profile aws-lz-dev-infraAdmin]
sso_start_url = https://d-93670ced1d.awsapps.com/start
sso_region = eu-west-1
sso_account_id = 906883863167
sso_role_name = InfraAdmin
region = eu-west-1
output = json
```
I added this as well..
```
cli_auto_prompt = on
```
...see https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-parameters-prompting.html
Checkout https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-alias.html

https://github.com/Flutter-Global/lzss-config-aws/tree/main/cdk/accounts-automation
https://github.com/tilfinltd/aws-extend-switch-roles

Install CDK and CDK assume role
```
11:54:11 matherp@MACC02X9DETJG5H in …/lzss-config-aws on 🌱mainon 🅰 (eu-west-1) ➜ sudo npm install -g cdk cdk-assume-role-credential-plugin fs-extra chalk
Password:
npm WARN deprecated querystring@0.2.0: The querystring API is considered Legacy. new code should use the URLSearchParams API instead.
npm WARN deprecated uuid@3.3.2: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.

added 38 packages, and audited 39 packages in 7s

3 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
npm notice
npm notice New minor version of npm available! 8.5.2 -> 8.6.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v8.6.0
npm notice Run npm install -g npm@8.6.0 to update!
npm notice
```

We will also need to authenticate to GitHub packages to install packages from private repositories, see https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry. For this we generate a Personal Access Token, see https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token, TLDR Settings > Developer Settings at the bottom > Personal Access Token > Generate New Token.

```
vi ~/.npmrc
cat ~/.npmrc
//npm.pkg.github.com/:_authToken=ghp_BLAHBLAHBLABH
```

To test I'm going to synth and diff an existing dev config by following...
https://flutteruki.zoom.us/rec/play/4e7H11v1xOrJL5GpxHsTAonJfgvOgI2V2eDsIomtOcenUEddBy_D-YG3TGYbW7Ss7-4DrdBLX2xa0fI3.INbeNQxh8jzSTnz8?continueMode=true

```
cd ~/Projects/src/github.com/
mkdir Flutter-Global
cd Flutter-Global/
git clone git@github.com:Flutter-Global/lzss-config-aws.git
cd lzss-config-aws/cdk/accounts-automation/
npm install

npm WARN deprecated source-map-url@0.4.1: See https://github.com/lydell/source-map-url#deprecated
npm WARN deprecated urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
npm WARN deprecated resolve-url@0.2.1: https://github.com/lydell/resolve-url#deprecated
npm WARN deprecated source-map-resolve@0.5.3: See https://github.com/lydell/source-map-resolve#deprecated
npm WARN deprecated sane@4.1.0: some dependency vulnerabilities fixed, support for node < 10 dropped, and newer ECMAScript syntax/features added
npm WARN deprecated querystring@0.2.0: The querystring API is considered Legacy. new code should use the URLSearchParams API instead.
npm WARN deprecated uuid@3.3.2: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.

added 765 packages, and audited 785 packages in 10s

80 packages are looking for funding
  run `npm fund` for details

1 high severity vulnerability

To address all issues, run:
  npm audit fix

Run `npm audit` for details.



# Later
https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-go.html
https://cdk8s.io/docs/latest/getting-started/
