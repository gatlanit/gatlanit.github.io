---
title: "Making a CLI Tool in C"
date: 2025-01-04T23:44:35-06:00
toc: false
images:
tags:
  - Programming
  - Computer Science
  - Music Production
---

I've always wondered how people make those commands you can type in your terminal like ```git``` and ```ffmpeg```. During my Winter Break from school, I decided to practice my C skills by learning how to make one.

{{< image src="/img/CommandExample.png" alt="A terminal with a binary run inside of it" position="center" style="border-radius: 3px;" >}}

## How are terminal commands made?
First, I should preface that by "commands", I am refering to binaries located in its 
respective ```$PATH``` directories in Linux and MacOS systems. 

Binaries are the compiled executables 
which allow you to run programs in the terminal like a command. In the case of C, usually 
you type in ./name when want to run an executable located in your working directory. 
If the binary is located instead in the /usr/local/bin folder (or whatever the output is 
when running ```echo $PATH```), then you no longer need the './' and can run the binary from 
anywhere in your computer.

## Part 1 - Making a simple C program
I decided that I wanted to make [a musical parameter generator](https://github.com/gatlanit/Song-Starter "Repo for my CLI tool") for my CLI tool. Basically a musical idea generator to have base for musicians to start a song off of. I figured that having things like a set BPM, Key, Mode, and Chord Progression would make it easier for musicians to get started with a track. I wanted this tool to not just be a simple toy project but something that would actually be useful.

#### The Plan

My plan was to start out with the easier parameters to generate before tackling randomizing the chord progression. For chord progressions, I didn't want to just have 4 or 8 random numbers, I wanted them to be at least somewhat musically coherent so I was going to deal with that later. For now, I got to work on randomizing BPM which was quite simple

```C
#include <stdlib.h>
#include <time.h>

int generate_bpm() {
  srand(time(0));
  return rand() % (200 - 75 + 1) + 75;
}
```
> ```srand(time(0))``` uses the system time as the seed for psuedo-randomization. Then, using ```rand()```, modulo it with a minimum of 75 BPM and a maximum of 200 BPM. the ```rand()``` function comes from the  C standard library ```stdlib.h```

Randomizing Key and Mode is also pretty straight forward, just create an array for possible values and use ```rand()``` to return a string from the selected element the array

```C
#include <stdlib.h>

char *generate_key() {
  static const char *keys[] = {"C",  "C♯", "D",  "D♯", "E",  "F",
                               "F♯", "G",  "G♯", "A",  "A♯", "B"};

  return (char *)keys[rand() % 12];
}

char *generate_mode() {
  static const char *modes[] = {"Major", "Minor"};

  return (char *)modes[rand() % 2];
}
```

Now it's time to tackle chord progression generation. After quite a bit of trial and error, I decided I would make an 2D array of possible chords for each chord in the progression before randomizing the length (either 4 or 8 bars long). Afterwards, I would loop through the length of the chord progression, picking a chord from the modes' respective array of possible chords. Finally appending each one and then printing it out. This was *by far* not the first thing I thought of, nor is it the best implementation, but for my purposes, it'll do.

```C
void generate_chord_progression(char *mode) {
  // Zeros added as placeholders to fill in allocated space

  /*
    Numbers 1-7 refer to each chord in a chord progression
    (e.g. 1 = Tonic, 5 = Perfect 5th, etc)
  */
  int majorChordChoices[8][6] = {{1, 6, 3, 1, 1, 6}, {4, 2, 5, 7, 0, 0},
                                 {5, 7, 4, 2, 1, 6}, {1, 6, 5, 0, 0, 0},
                                 {4, 2, 1, 6, 0, 0}, {5, 7, 6, 2, 6, 0},
                                 {5, 1, 6, 0, 0, 0}, {1, 6, 3, 0, 0, 0}};

  int minorChordChoices[8][6] = {{1, 6, 3, 1, 1, 6}, {4, 2, 5, 7, 0, 0},
                                 {5, 7, 4, 2, 1, 6}, {1, 6, 5, 0, 0, 0},
                                 {4, 2, 1, 6, 0, 0}, {5, 7, 4, 2, 6, 0},
                                 {5, 1, 6, 0, 0, 0}, {1, 6, 3, 0, 0, 0}};

  int columnSizes[8] = {6, 4, 6, 3, 4, 5, 3, 3}; // Predetermined number of columns for each row
  int progresionLengths[] = {4, 8}; // Either 4 or 8 bars long
  int length = progresionLengths[rand() % 2];

  int chordProgression[length];

  for (int i = 0; i < length; i++) {
    if (strcmp(mode, "Major") == 0) {
      int sizeOfColumn = columnSizes[i % 8];
      int randomIndex = rand() % sizeOfColumn;

      chordProgression[i] = majorChordChoices[i % 8][randomIndex];
    } else {
      int sizeOfColumn = columnSizes[i % 8];
      int randomIndex = rand() % sizeOfColumn;

      chordProgression[i] = minorChordChoices[i % 8][randomIndex];
    }
  }

  printf("Chord Progression: { ");
  for (int i = 0; i < sizeof(chordProgression) / sizeof(chordProgression[0]);
       i++) {
    printf("%d ", chordProgression[i]);
  }
  printf("}\n");
}
```

## Part 2 - Make it a command runnable in the terminal

As mentioned previously, it works when the binary is placed one of the 
locations from [```$PATH```](https://en.wikipedia.org/wiki/PATH_(variable) 
"Wikipedia page for Path"). I decided on ```/usr/local/bin``` since I know 
it works on both MacOS and Linux but you can pick which ever ones you want 
for your use case. Just make sure that it is a valid ```$PATH``` or you can
otherwise make a directory your desired path by appending it after a 
```:``` in the ```$PATH``` variable. Just note that if you're binary has 
the same name as another one, the one from the left most side of $PATH will take precedence. 

Now I decided to take some time to make a [bash script](https://github.com/gatlanit/Song-Starter/blob/master/install.sh) to automate this for me, that way it'll fetch the latest releases
from my repo. Then, adding an argument and calling an update method in my program will update it to the latest version
in my /usr/local/bin/ folder.

```C
void check_updates() {
  printf("Checking for updates...\n");
  int result = system("curl -sSL https://raw.githubusercontent.com/gatlanit/Song-Starter/master/install.sh | bash");

  if (result == 0) {
    printf("Update successful.\n");
  } else {
    printf("Update failed.\n");
  }
} 
```
> Because the [shell script](https://github.com/gatlanit/Song-Starter/blob/master/install.sh) is located directly in my repo, a simple curl command will run the shell script and update it accordingly. 

## Part 3 - Packaging and Distribution

Even though at this point it worked well enough manually uploading binaries to releases on GitHub, it gets a bit tiring having to go into a VM and make binaries for each platform 
(aarch64 and x86-64 for both MacOS and Linux). While looking up how other projects did this, I found out about [GitHub actions](https://github.com/gatlanit/Song-Starter/blob/master/.github/workflows/release.yml). In a nutshell, they are automations GitHub acts on upon a specific action. 

To make a GitHub action, you just need to make a ```.yml``` file in a ```.github/workflows``` folder and write in there. For me I wrote it in ```.github/workflows/release.yml```.
For [my program](https://github.com/gatlanit/Song-Starter/blob/master/.github/workflows/release.yml), I decided that that would be upon pushing tags to origin since I was already using Git for version control anyway.

```
git tag -a vX.X.X -m "Release version vX.X.X | Cool release!"
git push origin vX.X.X
```
---
# Conclusion
In the future I would like to add some more CLI arguments to my program and make it more customizable as to what parameters should be fixed and what shouldn't. But I figured musicians wouldn't bother to look at that anyway so I'll do that later. In the meantime, you can download my little program out on Linux and MacOS with a simple curl script:
```
curl -sSL https://raw.githubusercontent.com/gatlanit/Song-Starter/master/install.sh | bash
```

Overall, this was a fun project for me and I learned a lot about open source development. 
- Basic Continuous Deployment with github actions
- How binaries in your /bin/ folder allow you to run them from anywhere
- C Programming
- Distribution of open source software

I even made a [track](https://github.com/gatlanit/Song-Starter?tab=readme-ov-file#do-you-have-writers-block-do-you-struggle-to-come-up-with-ideas-for-music-this-program-might-be-able-to-help-you-out) in Ableton using paramaters from my program as a base!