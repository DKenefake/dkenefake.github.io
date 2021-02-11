---
layout: post
mathjax: true
title: Synthtic Guitar Strings - Kurplus-Strong Algorithm
date: 2021-01-30
category:
  - Blog
tags:
  - Music
  - Python
  - Code
---

This is just a fun post, where I look at the [Karplus-Strong Algorithm](https://en.wikipedia.org/wiki/Karplus%E2%80%93Strong_string_synthesis). This algorithm is used to produce synthetic guitar sounds via looping a waveform repeatedly thru a filter. That might sound strange, but what it is doing is taking a 'pluck' of random noise; you continuously feed it back into a filter that stands in as the 'string' decaying over time. This is what would make sense after you pluck a guitar string. It eventually becomes quiet. There are a few moving parts to this post, but you will get a treat at the end. 

The first thing we need to do is to build a data structure that loops back on itself and can be indexed basically forever. This is a standard data structure called a [Ring Buffer](https://en.wikipedia.org/wiki/Circular_buffer). The Wikipedia article has much more information and explanation than I can address here. This is a simple implementation of a Ring Buffer; we have overloaded the getters and setters for indexing the buffer. This allows us to index off of the current '0' position of the Ring Buffer. Here the populate buffer function fills the buffer with random data between $-.5$ and $.5$. This simulates a random pluck.

```python
class RingBuffer:
    
    def __init__(self, n: int, initial_vals = None):
        
        #set up buffer state
        
        self.size = n
        self.cursor = 0
        self.buffer = numpy.zeros(self.size)
        
        #set buffer values
        if initial_vals is not None:
            self.buffer[:] = inital_vals[0:self.size]
        else:
            self.populate_buffer()
            
    def next(self):
        
        # pushes the cursor forward one position
    
        self.cursor += 1
        
        if self.cursor >= self.size:
            self.cursor = 0
    
    def __getitem__(self, index:int):
        return self.buffer[(index + self.cursor + self.size)%self.size]
    
    def __setitem__(self, index:int, value):
        self.buffer[(index + self.cursor + self.size)%self.size] = value
    
    def populate_buffer(self):
        #fills buffer with random numbers in [-.5, .5]
        self.buffer[:] = numpy.random.rand(self.size)-.5
        
```

Now that we have that out of the way, we can implement the actual Karplus-Strong algorithm. This is actually quite simple to implement with the ring buffer already implemented. This is assuming a known sampling rate of your computer; in my case, it is 48000 Hz. The actual KS algorithm takes 1 line to implement. We take the next 2 values in the ring buffer, take their average, and then dampen it. Yep, That is it. 

```python
class KS_String:
    
    def __init__(self, freq:int, dampen:float = .999):
        self.freq = int(freq)
        self.ring_buffer = RingBuffer(sampling_rate // self.freq)
        self.dampen = dampen
        
    def get_value(self):
        # this is the heart of the KS algorithm
        self.ring_buffer[0] = .5*self.dampen*(self.ring_buffer[1] + self.ring_buffer[2])
        self.ring_buffer.next()
        return self.ring_buffer[-1]
    
    def pluck(self):
        self.ring_buffer.populate_buffer()
```

Now that we can pluck a string to a frequency, that means we can play a note. You already guessed it. We are making a note object. This note object takes a KS_String object and has a duration to it.

```python

class Note:
        
    def __init__(self, freq, duration:float = .25):
        
        # this is assuming 4/4 time
        # default note length is quarter note
        # defualt tempo is 120 bpm
        self.freq = int(freq)
        self.string = KS_String(self.freq)
        self.duration = duration
        self.num_steps = 60*4*duration*sampling_rate/bpm
        self.step_count = 0
        
    def get_value(self):
        self.step_count += 1
        
        if not self.isComplete(): 
            return self.string.get_value()
        else:
            return 0
    
    def isComplete(self):
        return self.step_count > self.num_steps
    
    def restart(self):
        # reset the note back to the start 
        self.string = KS_String(self.freq)
        step_count = 0
```

Now we have noted; we can make an instrument! This is a rather simple object that takes lists of notes and when the notes start. This assumes that the notes are so that they are played.

```python
class Instrament:
    
    def __init__(self, notes:list, starts:list):
        self.notes = notes
        self.starts = numpy.array([int(start*60*sampling_rate/bpm) for start in starts])
    
    
    def compose(self):
        
        # initalize the variables
        counter = 0
        sound = list()
        active_note = -1
        
        while True:
            
            # condition to switch to the next note
            if active_note != len(self.notes)-1:
                if counter == self.starts[active_note +1]:
                    active_note = active_note+1
                    print('new note')
            
            # play nothing if no note is active
            if active_note < 0:
                sound.append(0)
            else:
                sound.append(self.notes[active_note].get_value())
            
            # termination conditions all notes are off
            if self.notes[-1].isComplete():
                break
            # go to the next time step  
            counter += 1
        
        # reset all of out instraments notes
        for i in range(len(self.notes)):
            self.notes[i].restart()
        
        # return the instrament music as a numpy array
        return numpy.array(sound) 

```

Now that all of that work is out of the way, We can FINALLY PLAY A SONG. I will pick 'Good King Wenceslas' as it is relatively simple and one of the first 'real' songs I learned when I was in band. This song wasn't really made for the guitar, but that never stopped a guitar player. I transcribed the notes from broadly available sheet music. While the song is out of copyright, specific compositions of it are copyrighted.

Here 'no' is a helper function that maps the note names and beat duration to a Note object with the associate frequency. There seems to be a problem with the frequency in this table. As you can hear in the last segment of the song, the C->B->A transition doesn't sound quite right.

```python

sampling_rate = 48000
bpm = 240

def no(val:str, dur):
    freq_dict = {'C':261, 'D':293, 'E':329, 'F':349, 'G':391, 'A':440, 'B':493, 'high_C':526}
    return Note(freq_dict[val], duration=dur)

notes = [no('F', 1), no('F', 1), no('F', 1), no('G', 1),no('F', 1),no('F', 1),no('C', 2), no('D', 1),no('C', 1),no('D', 1),no('E', 1),no('F', 2),no('F', 2)]

#add refrain

notes.extend([copy.deepcopy(i) for i in notes])

#add the end to the song

finish = [no('high_C', 1), no('B', 1), no('A', 1), no('G', 1), no('A', 1), no('G', 1), no('F', 2), no('D', 1), no('C', 1), no('D', 1), no('E', 1), no('F', 2), no('F', 2)]

notes.extend(finish)

# calcuate to note start times
starts = (numpy.cumsum([i.duration for i in notes]) - 1).tolist()

guitar = Instrament(notes, starts)
music = guitar.compose()
```

This composed piece of music can then be turned into a sound file, and this is the result. WARNING LOUD!!!

<audio controls="controls">
  <source type="audio/wav" src="/assets/imgs/KingW.wav"></source>
  <p>Your browser does not support the audio element.<p>
</audio>
