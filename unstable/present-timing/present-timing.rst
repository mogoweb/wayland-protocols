.. Copyright 2020 Collabora, Ltd.

Wayland Present Timing Protocol Design Goals
============================================

TODO 

Example of using ideal_present_time to evaluate presentation rates
------------------------------------------------------------------

Assume a 60Hz fixed rate display and a client which has currently determined it
can only present smoothly every 66.6ms (i.e., at 15Hz), so it sets
target_present_time to T+66.6ms (T: target_present_time of previous commit).

At some point the client may want to check if it can present smoothly more
often. The client could check this by setting target_present_time to T+16.67ms
for a few frames, but doing so could lead to stutter if the +16.67ms deadline
couldn't be consistently met (which is what we are trying to evaluate).
Instead, the client keeps target_present_time at T+66.6ms and sets
ideal_present_time to T+16.67ms.

ideal_present_time tells the compositor to evaluate if it could have presented
at that point (or later), without actually doing so. The compositor informs the
client about the result of this evaluation using the optimal_present_time field
of the feedback event. So, for the case above, the feedback event would contain
an actual_present_time of T+66.6ms (assuming it was indeed still able present
at that point), and an optimal_present_time which is >=ideal_present_time and
<actual_present_time (i.e., T+16.67ms or T+33.3ms in this case), if the
compositor was able to present earlier, or equal to actual_present_time if the
compositor couldn't present earlier. This allows the client to evaluate
different presentation rates without visual anomalies.

Note that the compositor may take into consideration other factors, in addition
to the readiness of the presented content, when reporting an optimal present
time. For example, consider a VRR display and a client which currently presents
every 16.7ms. At some point the client may want to check if it can present at
half the rate, every 33.3ms, so it sets ideal_present_time to T+33.3ms.
Assuming the client's frame rendering requirements haven't changed, the
rendering should be ready every 33.3ms (since it's ready every 16.7ms).
However, the compositor may decide that the jump from 16.7ms to 33.3ms on the
VRR display may cause luminance artifacts, and thus may want to slowly ramp up
the presentation duration. Therefore it may present as optimal a time of
T+20ms, then a time of T+22ms etc.
