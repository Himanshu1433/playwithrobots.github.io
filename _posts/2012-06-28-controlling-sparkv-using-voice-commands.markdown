---
layout: post
title: "Controlling SparkV using voice commands"
category: "sparkV-Robot"
comments : true
redirect_from: "/working-on-robots/sparkv/17-controlling-sparkv-using-voice-commands/"
---
After controlling SparkV using [Android phone's accelerometer](/controlling-sparkv-using-android-phone-accelerometer), I decided to control it using voice commands. While searching for suitable tutorials, I found a very good one on [speech recognition and teleoperate TurtleBot Robot. ](http://www.pirobot.org/blog/0022/)The tutorial is very easy to understand and has made our task very easy. The prerequisite for this tutorial is basic understanding of [ROS](http://www.ros.org/ "ROS "), integrating [ROS and SparkV](/ros-and-sparkv "ROS and SparkV ") and a little about [python](http://python.org/). 

### Installation:

1. Follow this tutorial [speech recognition and teleoperate TurtleBot Robot](http://www.pirobot.org/blog/0022/) and understand how speech is recognized and how to add a vocabulary.
2. Now in the above tutorial after speech recognition, velocity messages are published using [<span style="font-family: 'courier new', courier;">geometry_msgs/Twist</span>](http://ros.org/doc/api/geometry_msgs/html/msg/Twist.html) message type, but we need this to be [<span style="font-family: 'courier new', courier;">sparkV_msg/velocity</span>](/ros-and-sparkv#velocity) in order to control sparkV. For this replace the code in <span style="font-family: 'courier new', courier;">voice_nav.py</span> file with the code mentioned below:

{% highlight python %}
#!/usr/bin/env python

"""
voice_nav.py allows controlling a mobile base using simple speech commands.
Based on the voice_cmd_vel.py script by Michael Ferguson in the pocketsphinx ROS package.
"""

import roslib; roslib.load_manifest('pi_speech_tutorial')
import rospy

from sparkV_msg.msg import velocity
from std_msgs.msg import String
from math import copysign
from std_msgs.msg import Bool

class voice_sparkV:
    def __init__(self):
        rospy.on_shutdown(self.cleanup)
        self.rate = rospy.get_param("~rate", 10)
        r = rospy.Rate(self.rate)
        self.paused = False

        # Initialize the Twist message we will publish.
        self.msg = velocity()
        self.msg2=Bool()
        
        # Publish the Twist message to the cmd_vel topic
        self.cmd_vel_pub = rospy.Publisher('motor', velocity)
        self.buzzer_pub = rospy.Publisher('buzzer', Bool)
        
        # Subscribe to the /recognizer/output topic to receive voice commands.
        rospy.Subscriber('/recognizer/output', String, self.speechCb)
        
        # A mapping from keywords to commands.
        self.keywords_to_command = {'stop': ['stop', 'freeze' ],
            'forward': ['straight'],
            'backward': ['backward'],
            'rotate left': ['rotate left'],
            'rotate right': ['rotate right'],
            'buzzer on': ['buzzer on'],
            'buzzer off': ['buzzer off','off'],
            }

        rospy.loginfo("Ready to receive voice commands")
        
        # We have to keep publishing the cmd_vel message if we want the robot to keep moving.
        while not rospy.is_shutdown():
            self.cmd_vel_pub.publish(self.msg)
            self.buzzer_pub.publish(self.msg2)
            r.sleep()
    
    def get_command(self, data):
        for (command, keywords) in self.keywords_to_command.iteritems():
            for word in keywords:
                if data.find(word) &gt; -1:
                    return command

    def speechCb(self, msg):
        command = self.get_command(msg.data)
        rospy.loginfo("Command: " + str(command))
        self.msg.s2L=True
        self.msg.s1L=True
        self.msg.s0L=True
        self.msg.s2R=True
        self.msg.s1R=True
        self.msg.s0R=True
        
        if command == 'forward':
            self.msg.left = True
            self.msg.right = True
            self.msg.start = True

        elif command == 'rotate left':
            self.msg.right = True
            self.msg.left = False
            self.msg.start = True

        elif command == 'rotate right':
            self.msg.right = False
            self.msg.left = True
            self.msg.start = True

        elif command == 'backward':
            self.msg.right = False
            self.msg.left = False
            self.msg.start = True

        elif command == 'stop':
            self.msg.start = False
            self.msg2.data = False

        elif command == 'buzzer off':
            self.msg2.data = False

        elif command == 'buzzer on':
            self.msg2.data = True

        else:
            return

def cleanup(self):
    # When shutting down be sure to stop the robot! Publish a Twist message consisting of all zeros.
    velocity = velocity()
    Bool=Bool()
    self.cmd_vel_pub.publish(velocity)
    self.buzzer_pub.publish(Bool)

if __name__=="__main__":
    rospy.init_node('voice_nav')
    try:
        voice_sparkV()
    except:
        pass
{% endhighlight %}

**3.**  My <span style="font-family: 'courier new', courier;">nav_commands.txt</span> file contains.  
{% highlight bash %}
buzzer on
buzzer off
move straight
move backward
rotate left
rotate right
freeze stop
{% endhighlight %}

**4.** Add the dependency of sparkV_msg package in <span style="font-family: 'courier new', courier;">manifest.xml</span> file of <span style="font-family: 'courier new', courier;">pi\_speech\_tutorial <span style="font-family: arial, helvetica, sans-serif;">package</span></span>.

**5.**  Make the pi\_speech\_tutorial package.
{% highlight bash %}
roscd pi_speech_tutorial
make
{% endhighlight %}

### Usage:
{% highlight bash %}
roscore

roslaunch pi_speech_tutorial voice_nav_commands.launch

roslaunch pi_speech_tutorial turtlebot_voice_nav.launch

rosrun rosserial_python serial_node.py /dev/ttyUSB0
{% endhighlight %}
<span>See the video of how it works :) . </span>

<div class="embed-responsive embed-responsive-16by9">
  <iframe class="embed-responsive-item" src="http://www.youtube.com/embed/XW8SS8Yy8bI"></iframe>
</div>
