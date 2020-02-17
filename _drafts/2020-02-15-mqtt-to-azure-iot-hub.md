---
layout: post
title:  "Secure MQTT communication with Azure IoT Hub"
date:   2020-02-17 21:32:00 +0100
categories: azure iot mqtt
---

I've started playing with the [IoT Hub in Azure][azure-iot-hub] and it took some time to assemble the pieces for testing secure [MQTT messaging][mqtt-homepage], so here's a writeup of some notes I made along the way.

{% include toc %}

# Getting started
First [sign up for a free Azure account][azure-free-account]. You'll get some free credits to play around with the services before committing to paying anything.

Then [install the Azure CLI][azure-cli]. Once that's installed, add the IoT extension:
{% highlight bash %}
az extension add --name azure-cli-iot-ext
{% endhighlight %}
And now you're ready to go!

# Create an IoT Hub
An instance of IoT Hub can be created using the Azure GUI (Create a resource -> search for IoT Hub) or using the CLI.

First create a Resource Group named "internet-of-gaz" to keep all the resources organised.
{% highlight bash %}
IOT_HUB_NAME=internet-of-gaz
az group create --name $IOT_HUB_NAME --location norwayeast
{% endhighlight %}

Next create a new IoT Hub instance with the same name. "norwayeast" is not available for IoT Hubs so we'll use "northeurope" instead.
{% highlight bash %}
az iot hub create --name $IOT_HUB_NAME --resource-group $IOT_HUB_NAME --location northeurope --partition-count 2
{% endhighlight %}

The output from the previous command should indicate that a new instance has been created under the "free tier". The [IoT Hub Pricing page][iot-hub-pricing] has more details about the differences between the tiers.

# Add a device
Now that we've got our hub we need to start registering our IoT devices. This registration process can be automated which is useful if you're dealing with a large number of devices, but let's start by registering a single device named "gaz-device-1".
{% highlight bash %}
IOT_DEVICE_NAME=gaz-device-1
az iot hub device-identity create --device-id $IOT_DEVICE_NAME --hub-name $IOT_HUB_NAME
{% endhighlight %}

And now we're ready for our device to start communicating with the hub!

# Monitoring Hub Messages
First we need a way to monitor messages received by the Hub. The [documentation has examples][iot-hub-default-endpoint] for how to use the SDKs to read from the default endpoints, but there's also a built-in monitor using the CLI:
{% highlight bash %}
az iot hub monitor-events -n $IOT_HUB_NAME
{% endhighlight %}

# Device to Cloud Messaging
I have the [Mosquitto broker software][mosquitto-homepage] installed, so I'll use [mosquitto_pub][mosquitto-pub-man] for sending messages to the Hub. To find out what to specify for the hostname, username, etc, we once again need to [Read The Fabulous Manual][iot-hub-mqtt-usage] to see what IoT Hub expects. The hostname, user, and topic are defined as follows:
{% highlight bash %}
MQTT_HOSTNAME=$(az iot hub show -n $IOT_HUB_NAME | jq -r .properties.hostName)
MQTT_USER=$MQTT_HOSTNAME/$IOT_DEVICE_NAME
MQTT_TOPIC="devices/$IOT_DEVICE_NAME/messages/events/"
{% endhighlight %}

Since we're using Mosquitto we're also going to need to obtain the PEM-encoded Azure CA certificates to enable SSL communication. This can be done by opening https://portal.azure.com in your browser, viewing the certificate details, and then (if using Firefox) click on the "Download PEM (cert)" link on the "Baltimore CyberTrust Root" tab. Save the file as "portal-azure-com.pem".

But what about the password for authentication? We'll look at two options: short-lived tokens and X.509 certificates.

## Shared Access Signature (SAS) Token Authentication
By default the device is registered using the "symmetric key" authentication type. This means that the messages from our device will need to include a "Shared Access Signature" (SAS) token for authentication. The [IoT Hub Security documentation][iot-hub-security] has examples of how to generate these tokens programatically using the SDKs. Alternatively we can use the CLI to generate a token that will be valid for 10 minutes (600 seconds). I haven't touched on "Shared access policies" here, but by default the token will use the "iothubowner" policy name, which isn't recommended for use in a production setting.

{% highlight bash %}
MQTT_PASS=$(az iot hub generate-sas-token -n $IOT_HUB_NAME -d $IOT_DEVICE_NAME --du 600 | jq -r .sas)
{% endhighlight %}

Putting it all together we get:
{% highlight bash %}
mosquitto_pub \
--cafile portal-azure-com.pem \
-h "$MQTT_HOSTNAME" \
-u "$MQTT_USER" \
-P "$MQTT_PASS" \
-i "$IOT_DEVICE_NAME" \
-t "$MQTT_TOPIC" \
-m "A message sent at: $(date)" \
-d
{% endhighlight %}

If we look in the monitoring window we should see the message appear:
{% highlight bash %}
Starting event monitor, use ctrl-c to stop...
{
    "event": {
        "origin": "gaz-device-1",
        "payload": "A message sent at: Sun 17 Feb 20:16:42 CET 2020"
    }
}
{% endhighlight %}

Hooray!

## X.509 Certificate Authentication
SAS tokens are fine if our device is able to generate its own tokens in some way, but that may not be possible. The device may not have the capacity or capability to perform the generation, so an alternative means of securing communications is by using [X.509 certificates][x509-wikipedia]. The basic idea here is that the device carries a certificate which is signed by a known and trusted Certificate Authority. These certificates may be generated installed on the devices at the time the device is manufactured, or distributed in a secure way. There's [an interesting Azure blog post][azure-x509-options] that covers some of the options here.

### Create a Certificate Authority
For our testing we're going to create our own Certificate Authority (CA) (referred to as "self-managed PKI" in that blog post) using good old openssl. This is as easy as the following steps:

1. Create a CA private key.
1. Create a CA certificate using the private key.
1. Convert the CRT to PEM format.
1. Upload the PEM to our IoT Hub.

{% highlight bash %}
# 1. Generate the CA key
openssl genrsa -des3 -out ca.key 2048                    

# 2. Generate the CA certificate
openssl req -new -x509 -days 1826 -key ca.key -out ca.crt

# 3. Convert the crt to pem format
openssl x509 -in ca.crt -out ca.pem

# 4. Upload the cert to our IoT Hub
az iot hub certificate create --hub-name $IOT_HUB_NAME --name GazCert --path ./ca.pem

{% endhighlight %}

### Proof of Posession
But if you list our ceritificates or view them in the Portal in our browser, you might notice that the certificate is listed as "not verified"
{% highlight bash %}
az iot hub certificate list --hub-name $IOT_HUB_NAME | jq '.value [].properties.isVerified'
false
{% endhighlight %}

This is because we need to perform what is known as "proof of posession", to prove that we actually control the CA. Once again there's [an Azure blog post detailing the steps involved][azure-proof-of-posession]. What it basically boils down to is using our CA to generate a certificate with a specific Common Name (CN) and then uploading the verification certificate into the Hub.

{% highlight bash %}
# 1. Get the "Entity Tag" of the CA cert uploaded earlier
az iot hub certificate list --hub-name $IOT_HUB_NAME | jq -r '.value [].etag'
AAAAAAt39Kw=

# 2. Get the verification code required for proof of posession
az iot hub certificate generate-verification-code --hub-name $IOT_HUB_NAME --name GazCert --etag AAAAAAt4EV8= | jq -r .properties.verificationCode
915F075941302270B6C1288DC4565DAB909FCC9A55E82E4D

# 3. Create verification key
openssl genrsa -out verification.key 2048

# 4. Create verification cert (use the verification code from step 2 as the Common Name)
openssl req -new -key verification.key -out verification.csr

# 5. Sign the certificate
openssl x509 -req -in verification.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out verificationCert.pem -days 1024 -sha256

# 6. Finally, upload the proof of posession
az iot hub certificate verify --etag AAAAAAt4El8= --name GazCert --path ./verificationCert.pem --hub-name $IOT_HUB_NAME
{% endhighlight %}

Repeat the command from earlier to ensure that the CA "isVerified" property is now true.

## Generate a Device Certificate
Now that we have a verified CA any certificates given to our devices will be accepted. Here we'll set up a new device and configure it to use "X509 CA signed" authentication.

First register a second device in IoT Hub and configure it to use X509 CA authentication:
{% highlight bash %}
IOT_DEVICE_NAME=gaz-device-2
az iot hub device-identity create --device-id $IOT_DEVICE_NAME --hub-name $IOT_HUB_NAME --auth-method x509_ca
{% endhighlight %}

Now generate a certificate for the device to use, and sign it using our CA.
{% highlight bash %}
# 1. Create client private key
openssl genrsa -out device2.key 2048

# 2. Create certificate request signed with private key (NOTE: The Common Name must be the name of the IoT device: gaz-device-2)
openssl req -new -out device2.csr -key device2.key

# 3. Complete the request and create a client cert:
openssl x509 -req -in device2.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out device2.crt -days 360
{% endhighlight %}

Now we're ready to send the message. Note that we no longer need to include 

{% highlight bash %}
# Redefine user and topic to pick up the new device name
MQTT_USER=$MQTT_HOSTNAME/$IOT_DEVICE_NAME
MQTT_TOPIC="devices/$IOT_DEVICE_NAME/messages/events/"

mosquitto_pub \
--cafile portal-azure-com.pem \
--cert client.crt \
--key client.key \
-h "$MQTT_HOSTNAME" \
-u "$MQTT_USER" \
-i "$IOT_DEVICE_NAME" \
-t "$MQTT_TOPIC" \
-m "x509 message at $(date)" \
-d
{% endhighlight %}

Then, as when sending the message using SAS authentication, the message should appear in the monitor window!

# Cleanup
When you're done playing with your IoT Hub then remember to remove the resources. If you're using the 'free tier' then you probably don't have to worry about an unexpected bill, but if you chose a Basic or Standard then delete the resource group and all the associated resources using the following command:
{% highlight bash %}
az group delete -n $IOT_HUB_NAME                                                                                  
{% endhighlight %}

Check the list of resource groups (using jq to cleanup the JSON output)
{% highlight bash %}
az group list | jq '.[].name' 
{% endhighlight %}

# Conclusion / What's Next?
Phew. That felt like quite a lot of work. So, to recap what we've achieved here:
1. Created a new IoT Hub in Azure
2. Registered a couple of devices, one for SAS and one for X.509 CA authentication
3. Generated a SAS token and used it to send an MQTT message
4. Created a CA, verified it with Azure, then generated a client certificate and using it to send an MQTT message

Now that we've got our events (messages) successfully flowing in to the Hub we need to decide what to do with them. Topics for future investigation include storage, monitoring, and analytical worktools. Azure has a shedload of native services to offer (DataBlocks, EventHub, databases, etc.), and of course it's pretty trivial to just spin up the infrastructure to run any other [open-source data engineering projects][oss-engineering] that might be useful. Stay tuned!


[azure-iot-hub]: https://azure.microsoft.com/en-us/services/iot-hub/
[mqtt-homepage]: http://mqtt.org/
[azure-free-account]: https://azure.microsoft.com/en-us/free/
[azure-cli]: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest
[iot-hub-pricing]: https://azure.microsoft.com/en-gb/pricing/details/iot-hub/
[jq-homepage]: https://stedolan.github.io/jq/
[iot-hub-security]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-security#security-tokens
[iot-hub-default-endpoint]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-messages-read-builtin
[mosquitto-homepage]: https://mosquitto.org/
[mosquitto-pub-man]: https://mosquitto.org/man/mosquitto_pub-1.html
[iot-hub-mqtt-usage]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-mqtt-support
[x509-wikipedia]: https://en.wikipedia.org/wiki/X.509
[azure-x509-options]: https://azure.microsoft.com/en-gb/blog/installing-certificates-into-iot-devices/
[azure-proof-of-posession]: https://docs.microsoft.com/en-us/azure/iot-dps/how-to-verify-certificates
[oss-engineering]: https://github.com/gunnarmorling/awesome-opensource-data-engineering