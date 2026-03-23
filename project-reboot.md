

# Reboot Kivy


Kivy in general is based too much on SDL and that python / EventLoop needs to drive.

Window is also bound to only work as a singleton, and should allow more instances to be spawned.

Also means that the Window no longer is our Provider and App itself should be that instead..

Clock also needs to work abit different and allow more finegrained timings that what clock pulses from fps can offer.


# Phase 0 - Prework to test some theories.

atm the Clock.tick concept works, but it keep adding extra frametime on top, so instead of 1 / 60 it comes 1 / 30..

since swift is driving the clock, Clock.tick doesnt seem to need to calculate any timings or do any sleeping..

it just needs to add current delta time to total time, or whatever drives Clock.schedule_interval callbacks to trigger at right timings..

i am wondering if we could use async Task AsyncSequence/ Task.sleep for this
that allows to setup some different groups, that you can attach 2..
soo you register different timings with some id 
afterwards you can jack into those different timings by id, and append a callback to it..
and you like before get an event class you later can cancel

or litterly pick the group and tell it to clear all callbacks..

we should still utilize EventLoop but for now it doesnt seem to serve us much purpose
because EventLoop or Clock should no longer be what calls the canvas_draw or draw_on in Thor Implementation so far... so let it remain in abit of dead state for now..

in general what drives canvas drawing updates, should run just based on timing of DisplayLink (and what ways the other platforms will preform it)

and Clock should run independent from DisplayLink with as mentioned above, run seperate Task with different sleep timings..

