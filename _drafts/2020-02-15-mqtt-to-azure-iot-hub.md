---
layout: post
title:  "Secure MQTT communication with Azure IoT Hub"
date:   2020-02-15 20:32:00 +0100
categories: azure iot mqtt
---
I've started playing with the [IoT Hub in Azure][azure-iot-hub] and it took some time to assemble the notes for testing secure [MQTT messaging][mqtt-homepage], so I thought it might be useful for someone else if I put these notes in one place.

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
az group create --name internet-of-gaz --location norwayeast
{% endhighlight %}

Next create a new IoT Hub instance with the same name. "norwayeast" is not available for IoT Hubs so we'll use "northeurope" instead.
{% highlight bash %}
az iot hub create --name internet-of-gaz --resource-group internet-of-gaz --location northeurope --partition-count 2
{% endhighlight %}

The output from the previous command should indicate that a new instance has been created under the "free tier". The [IoT Hub Pricing page][iot-hub-pricing] has more details about the differences between the tiers.

# Add a device
Now that we've got our hub we need to start registering our IoT devices. This registration process can be automated which is useful if you're dealing with a large number of devices, but let's start by registering a single device named "gaz-device-1".
{% highlight bash %}
az iot hub device-identity create --device-id gaz-device-1 --hub-name internet-of-gaz
{% endhighlight %}

And now we're ready for our device to start communicating with the hub!

# Monitoring the Hub Messages
First we need a way to monitor messages received by the Hub. Once again the documentation has the answer. Here's trimmed down version of the example using the Python SDK to monitor the default built-in endpoint:

{% highlight python %}
import os
from azure.eventhub import EventHubConsumerClient

CONNECTION_STR = os.environ["EVENT_HUB_CONN_STR"]
EVENTHUB_NAME = os.environ['EVENT_HUB_NAME']

def on_event(partition_context, event):
    print("Event: {}".format(event))

if __name__ == '__main__':
    consumer_client = EventHubConsumerClient.from_connection_string(
        conn_str=CONNECTION_STR,
        consumer_group='$Default',
        eventhub_name=EVENTHUB_NAME,
    )

    try:
        with consumer_client:
            consumer_client.receive(
                on_event=on_event,
                starting_position="-1",  # "-1" is from the beginning of the partition.
            )
    except KeyboardInterrupt:
        print('Stopped receiving.')
{% endhighlight %}


# Generate a SAS Token
By default the device is registered using the "symmetric key" authentication type. This means that the messages from our device will need to include a "Shared Access Signature" (SAS) token for authentication. The [IoT Hub Security documentation][iot-hub-security] has examples of how to generate these tokens programatically using the SDKs. Alternatively we can use the CLI to generate a token that will be valid for 5 minutes (300 seconds). I haven't touched on "Shared access policies" here, but by default the token will use the "iothubowner" policy name, which isn't recommended for use in a production setting.

{% highlight bash %}
az iot hub generate-sas-token -n internet-of-gaz -d gaz-device-1 -du 300
{% endhighlight %}

# Device to Cloud Messaging
So let's try sending a message to the cloud
 az iot hub generate-sas-token -n internet-of-gaz
{
  "sas": "SharedAccessSignature sr=internet-of-gaz.azure-devices.net&sig=eypv1%2FOIzXftGGgKn9f%2FspoTVCgGrhU6cisjo%2B8l44s%3D&se=1581442973&skn=iothubowner"
}
export IOT_HUB_HOST="internet-of-gaz.azure-devices.net"
export IOT_HUB_DEVICE="device2"
export PASS=$(az iot hub generate-sas-token -n internet-of-gaz -d device2 | jq -r .sas)

mosquitto_pub \
--cafile portal-azure-com.pem \
-h "$IOT_HUB_HOST" \
-u "$IOT_HUB_HOST/$IOT_HUB_DEVICE" \
-P $PASS \
-i "$IOT_HUB_DEVICE" \
-t "devices/$IOT_HUB_DEVICE/messages/events/" \
-m "$(date)" \
-d

(port not necessary: 8883 default)




Subscribe to the events
https://devblogs.microsoft.com/iotdev/read-azure-iot-hub-device-to-cloud-messages-from-the-built-in-and-custom-endpoints/

https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-messages-read-builtin




Generate self-signed CA and cert
http://www.steves-internet-guide.com/creating-and-using-client-certificates-with-mqtt-and-mosquitto/
1. Create CA key: openssl genrsa -des3 -out ca.key 2048
2. Create CA cert using key: openssl req -new -x509 -days 1826 -key ca.key -out ca.crt



"Proof of Possession"
1. Create verification key: openssl genrsa -out verification.key 2048
2. Create verification cert: openssl req -new -key verification.key -out verification.csr
	Ensure common name is username?
3. Create proof of posession cert:

openssl x509 -req -in verification.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out verificationCert.pem -days 1024 -sha256

Create a client cert for your device
3. Create client private key: openssl genrsa -out client.key 2048
4. Create cert request signed with private key: openssl req -new -out client.csr -key client.key
	Ensure common name is username?
5. Complete the request and create a client cert: openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 360
openssl x509 -in cert.crt -out cert.pem.
	can we skip this by changing to -out client.pem? 


    ca.crt – The Certificate authority certificate
    client.crt – The client certifcate file
    client.key – The client private key


Upload to IoT Hub
https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-security-x509-get-started


mosquitto_pub \
--cafile portal-azure-com.pem \
--cert client.crt \
--key client.key \
-h "$IOT_HUB_HOST" \
-u "$IOT_HUB_HOST/$IOT_HUB_DEVICE" \
-i "$IOT_HUB_DEVICE" \
-t "devices/$IOT_HUB_DEVICE/messages/events/" \
-m "$(date)" \
-d

# Cleanup
When you're done playing with your IoT Hub then remember to remove the resources. If you're using the 'free tier' then you probably don't have to worry about an unexpected bill, but if you chose a Basic or Standard then delete the resource group and all the associated resources using the following command:
{% highlight bash %}
az group delete --name internet-of-gaz2                                                                                  
{% endhighlight %}

Check the list of resource groups (using jq to cleanup the JSON output)
{% highlight bash %}
az group list | jq '.[].name' 
{% endhighlight %}

# Other useful links
1. https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-security
2. https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-security-x509-get-started


[azure-iot-hub]: https://azure.microsoft.com/en-us/services/iot-hub/
[mqtt-homepage]: http://mqtt.org/
[azure-free-account]: https://azure.microsoft.com/en-us/free/
[azure-cli]: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest
[iot-hub-pricing]: https://azure.microsoft.com/en-gb/pricing/details/iot-hub/
[jq-homepage]: https://stedolan.github.io/jq/
[iot-hub-security]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-security#security-tokens