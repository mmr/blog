---
layout: post
title: "Perseverance"
date: 2014-04-21 04:29:50 +0000
comments: true
categories: [cheat, games, programming, java, wsh, c#]
---

Some weeks ago I bought the _SEGA Humble Bundle_.
Along with some good old classics that surely deserves posts on their own -like
_Altered Beast_, _Golden Axe_ and the fantastic [The Typing of The Dead: Overkill](http://store.steampowered.com/app/246580/)-
was [Hell Yeah! Wrath of the Dead Rabbit](http://store.steampowered.com/app/205230/).

## Sonic on Steroids!
{% img /images/hell-yeah-splash.jpg 1024 800 'Sonic on Steroids!' %}
Hell Yeah is a crazy action-adventure platformer, full of cartoon violence and toilet humor.
Instant fun for all ages!
Do not let the somewhat low _metacritic_ score fool you. Hell Yeah! is **crazy** good.

For days I have played it during my lunch break, and when it was over and the final boss was put out of his misery
I wanted more, I wanted to get 100% achievements!

There are currently 12 achievements in the game.
After a couple hours 11 were down, just 1 left to go.
But that last one proved to be hard to get...

## The Jackpot
You have to hit **JACKPOT** (777) in the _Cassino_ stage slot machines.
And there I was, pressing the '**E**' key over and over, repeated times...
Pretending it was fun to watch the symbols roll and then, after 10 minutes
into this very tedious task I finally decided to do what every sane
person in similar grounds would do... Break the rules! I mean, write a program!

## <s>Adventure</s> Programming Time!
Well, "how hard can this be" I thought.
All the program has to do is press the '**E**' key like there is no tomorrow.
Pausing once in a while so the game can process the command.

I was on my younger sons' Windows 7 laptop, with no dev tools.
Despite this harsh, non-dev-friendly environment, I was convinced that this
tale would end quickly and bed would see a happier me with 100% achievements
soon. I could not be more wrong...

### Windows Script Turn
What is the quickest way to get this done? [WSH](http://en.wikipedia.org/wiki/Windows_Script_Host), I choose you!
``` vb.net Gotta love the simplicity imbued in all forms of scripting...
' Create shell object that can "type" keys
Set s = WScript.CreateObject("WScript.Shell")

Do
  ' Focus the correct window and send the key
  if s.AppActivate("Hell Yeah!") Then
    s.SendKeys "e"
  End If

  ' Giving us some time in case we want to kill the script
  WScript.Sleep 2000
Loop
```

Everything was going smoothly according as planned.
It was simple and beautiful, so beautiful that a tear of pure joy started to take form
in the corner of my left eye and then... nothing...

The game window was getting focused but nothing happened, no roll in the slot machine, just resounding silence.
The '**E**' key was not being pressed, or at least not in a way the game would understand.

By that time my wife was already asking me to go bed, but I was so close, or so I thought...
It would be a shame to give up now. And started downloading **Vim** and the **JDK**...

### Java Turn
Usually, Java should be far from the top if your main concern is getting things done fast.
But since we have **java.awt.Robot**, which sole purpose is to solve problems like ours, guess
that changes the picture.

``` java What was that word again? Oh yes, verbosity...
import java.awt.Robot;
import java.awt.event.KeyEvent;

public class Jackpot {
  public static void main(String[] args) throws Exception {
    Robot bot = new Robot();

    // Giving us some time to focus the correct window manually
    Thread.sleep(5000);

    // Press and Release the E key 
    // Give the game some time to process the key press
    while (true) {
      for (int i = 0; i < 20; i++) {
        bot.keyPress(KeyEvent.VK_E);
        bot.keyRelease(KeyEvent.VK_E);
        Thread.sleep(500);
      }

      // Give us some time to kill the program
      Thread.sleep(1000);
    }
  }
}
```

The key was being sent, I could see it but again, nothing happened in the game.
I was so optimistic about Robot, it helped me in the past, did not expect it to fail me now.

My wife fell asleep watching classic Pink Panther cartoons.
Fixed the blanket and kissed her good night. I had a mission to accomplish!

We were getting into the late hours, further into the night.
Cats fighting and hounds barking could sporadically be heard on the streets, time to master
focus and boost ninja problem solving mad-skills:

  - **Headphones**: _Check_
  - **Good Music**: _Check_
  - **Energy Drink**: _Check_ _Check_ _Check_

Nothing can stop me now, I will get this done!

Think! What is wrong? Why is the game not getting the keypress events?

What about sending the keys in raw directly to Win32 API?
Calling _SendInput_ from _user32.dll_ should do it.
The options are JNI, C/C++ and C#.
Started downloading *.NET SDK* and [Xamarin](http://en.wikipedia.org/wiki/Xamarin).

### .NET Turn
After reading several pages on [MSDN](http://msdn.microsoft.com/en-us/library/windows/desktop/ms646310(v=vs.85\).aspx) and
related forums about how the _SendInput_ function and the **INPUT** data structure works, things started making sense.

Collected several pieces of code and experimentations and came up with the following beast:
``` c# Hey little fly, get ready to be <s>squashed</s> cannon balled! Rigmarole FTW!
using System;
using System.ComponentModel;
using System.Runtime.InteropServices;
using System.Threading;

namespace Jackpot
{
  class Jackpot
  {
    struct INPUT
    {
      public INPUTType type;
      public INPUTUnion Event;
    }

    [StructLayout (LayoutKind.Explicit)]
    struct INPUTUnion
    {
      [FieldOffset (0)]
      internal MOUSEINPUT mi;
      [FieldOffset (0)]
      internal KEYBDINPUT ki;
      [FieldOffset (0)]
      internal HARDWAREINPUT hi;
    }

    [StructLayout (LayoutKind.Sequential)]
    struct MOUSEINPUT
    {
      public int dx;
      public int dy;
      public int mouseData;
      public int dwFlags;
      public uint time;
      public IntPtr dwExtraInfo;
    }

    [StructLayout (LayoutKind.Sequential)]
    struct KEYBDINPUT
    {
      public short wVk;
      public short wScan;
      public KEYEVENTF dwFlags;
      public int time;
      public IntPtr dwExtraInfo;
    }

    [StructLayout (LayoutKind.Sequential)]
    struct HARDWAREINPUT
    {
      public int uMsg;
      public short wParamL;
      public short wParamH;
    }

    enum INPUTType : uint
    {
      INPUT_KEYBOARD = 1
    }

    [Flags]
    enum KEYEVENTF : uint
    {
      EXTENDEDKEY = 0x0001,
      KEYUP = 0x0002,
      SCANCODE = 0x0008,
      UNICODE = 0x0004
    }

    const short KEY_E = 0x0012;

    [DllImport ("user32.dll")]
    static extern UInt32 SendInput (int numberOfInputs, INPUT[] inputs, int sizeOfInputStructure);

    private static void SendKey (short key)
    {
      // create input events as unicode with first down, then up
      INPUT[] inputs = new INPUT[2];
      inputs [0].type = inputs [1].type = INPUTType.INPUT_KEYBOARD;
      inputs [0].Event.ki.dwFlags = inputs [1].Event.ki.dwFlags = KEYEVENTF.SCANCODE;
      inputs [0].Event.ki.wScan = inputs [1].Event.ki.wScan = key;    
      inputs [1].Event.ki.dwFlags |= KEYEVENTF.KEYUP;
      SendInput (inputs.Length, inputs, Marshal.SizeOf (typeof(INPUT)));
    }

    static void Main (string[] args)
    {
      // Giving us some time to focus the correct window manually
      System.Threading.Thread.Sleep (5000);

      while (true) {
        for (int i = 0; i < 20; i++) {
          SendKey (KEY_E);
          Thread.Sleep (500);
        }
        Thread.Sleep (2000);
      }
    }
  }
}
```

Yes, that is a lot more code than I thought was necessary to emulate a simple keyPress using the Win32 API and guess what... it does **NOT** work.
The reasons why this does not work are beyond me but I bet DirectInput is to blame.

No more cat fights or barks, dawn birds started singing instead, enunciating the first sheds of sunlight.

My hope was beaten to a pulp.
I was tired, very tired and suddenly noticed I had not eaten for the past several hours.
Headaches started consuming me... Giving up crossed my mind but not before one last shot.
I can do this!

### c# round two
Light could be seen at the end of tunnel. Hopefully it was not the train.

[Interceptor](https://github.com/jasonpang/Interceptor) promissed to simulate keystrokes to _DirectX_ games.
Followed the installation instructions and wrote some code:
``` c# Nothing like loading a kernel module to heat things up...
using Interceptor;
using System.Threading;

namespace Jackpot
{
  public class Jackpot
  {
    private static Input input = new Input();

    public static void Main (string[] args)
    {
      // Give us some time to focus the correct window manually
      Thread.Sleep (5000);

      // Load driver
      input.KeyboardFilterMode = KeyboardFilterMode.All;
      input.KeyPressDelay = 10;
      input.Load();

      while (true) {
        for (int i = 0; i < 20; i++) {
          input.SendKeys (Keys.E);
          Thread.Sleep (500);
        }
        Thread.Sleep (1000);
      }
    }

    ~Jackpot() {
      input.Unload ();
    }
  }
}
```

Marcos, my younger boy, the proud owner of the laptop I had recently transformed into a powerhouse development station,
woke up. Just in time to hear the fantastic story of how his daddy spent a whole night trying to cheat in a game.

"Marcos, of course cheating is **wrong**. I did it out of curiosity! You see, knowing how to bend the rules does **not**
mean you absolutelly have to. In fact, you can help make the world a better place if you know about this kind of stuff! (...)"

There was a slight contemplative period of silence, while he looked puzzled to the slot machine being rolled with no
human interaction, as if by **Magic**!

After approximately 50 minutes and 5500 **E** key presses... **JACKPOT**!!
{% img /images/hell-yeah-jackpot.jpg 1024 800 'To the victor go the spoils' %}
{% img /images/hell-yeah-all-achievements.png 1024 800 '100% achievements... I can finally rest in peace' %}

"Daddy, why didn't you build a _Homer Drinking Bird_?" - He asked

"What is that?" - I replied

{% img /images/homer-drinking-bird.png %}

Brilliant! :)

