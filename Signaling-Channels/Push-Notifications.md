

# Privacy Analysis of Push Notifications for Anti-censorship

Push notifications are a cost-effective, performant, and censorship-resistant way for application service providers to send messages to users[0], but come with a unique set of privacy risks. This is a privacy analysis of the use of push notifications to deliver updates to suggested anti-censorship settings.

In our design we completely mitigate the following risks:
- We do not leak metadata about users' activity, including Tor usage patterns
- We do not store information that can be used to either uniquely identify or filter out subgroups of users
- We do not leak guard information about users
- We do not expose users to a mitm attack that would allow an adversary to control or suggest their entry point to the Tor network

## Background on Push Notifications

Push notifications rely on a provider, such as Google's Firebase Cloud Messenger (FCM), to forward messages from a subscribing service directly to users. Clients that have push notifications enabled maintain a background connection to the push notification provider. For each registered service, they generate a new token and send that directly to the service provider. When the service provider wishes to send a message to the client, they send this token along with the message to the push notification provider, which then directs the message through the long-lived background connection to the intended recipient.

The subscribing service must store the registration token of the user, to be sent to the push notification provider along with the message.

## Privacy risks

The background connection between a user's device and push notification service presents a risk to privacy. The changes in a user's IP address could potentially be used to track the user's movements and whether they are online. However, the vast majority of Android users will have Google services installed and will already have this background connection to the push notification provider. We will focus mostly on the additional privacy risk of using push notifications with Tor applications.

### Preventing the leakage usage patterns

One of our primary concerns is to not leak the Tor usage patterns of users to either the push notification provider or our anti-censorship settings service. To this purpose, messages sent through push notifications must never be triggered as a result of user behaviour. They should only be sent in response to changes or information that affect all users equally, which in this case is would be updates to our recommended anti-censorship settings. These updates happen in response to censorship events, the addition of new features for anti-censorship tools, or changes in bridge availability.

### Not storing uniquely identifying information

For push notifications to work, the minimum amount of information we need to store as the service provider are the push notification tokens of registered users. Another main concern of ours is to prevent an adversary from using any additional information stored at our anti-censorship service to uniquely identify users and get their registration tokens. To prevent this, we do not store any additional information such as the time of registration, IP addresses, or location information. We originally contemplated storing the user's country code in order to provide location-specific settings suggestions, but this could just as easily be accomplished locally by sending to all users a mapping of locations and settings.

### Authentication of circumvention settings

Another major risk is that the push notification provider can modify the contents of the anti-censorship settings map updates to influence a specific user's entry point to the Tor network. This would pose a significant threat to the user's anonymity. We fully mitigate this by signing all messages from our push notification service. Our public key is shipped with the Android application and used to verify the authenticity of the settings.

### Protecting user bridge information

The circumvention settings that we send over push notifications are all public and available at https://bridges.torproject.org/moat/circumvention/map, however we additionally encrypt the contents of the push notification messages. If we expand the use-case of this push notification channel to include specific bridges from rdsys, this will be critical to prevent information leaks about which bridges users are likely connecting through.

## References

[0] https://www.petsymposium.org/foci/2023/foci-2023-0009.pdf  
CenPush paper: https://petsymposium.org/popets/2025/popets-2025-0153.pdf