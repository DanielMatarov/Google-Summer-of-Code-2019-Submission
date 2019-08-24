## **Interactive Music - Daniel Matarov**

-   **Project:**  Interactive Music for Godot Engine
-   **Student:**  Daniel Matarov ([DanielMatarov](https://github.com/DanielMatarov))
-   **Mentors:**  Juan Linietsky ([reduz](https://github.com/reduz))
-   **Repository:**  [https://github.com/DanielMatarov/godot/tree/Interactive-Music/modules/InteractiveMusic](https://github.com/DanielMatarov/godot/tree/Interactive-Music/modules/InteractiveMusic)

## Project Overview
The aim of this project was to implement an interactive music feature to Godot Engine, which consists of adding tempo and beat functinalities to existing AudioStreams and adding two new classes, called AudioStreamPlaylist and AudioStreamTransitioner. 
The new BPM functionalities work by setting a BPM to an AudioStream, and transitioner and playlist are able to count the samples with accuracy, the number of which depending on how many beats the user has input, either on a stream in Playlist, or fades in Transitioner. 
AudioStreamPlaylist can play multiple AudioStreams in a sequence, based on their tempo and beats. The class has options for looping and shuffling the clips, and can take up to 64 clips. 
The way the code works is by calculating the lenght in samples of a stream's beat count, based on their or the playlist's default tempo, and processes them in small blocks. When the frames are all processed, the next stream's samples are calculated, and the process starts again, then the previous stream has a brief and unnoticable fade out while the next stream starts. This allows for accurate time keeping and seamles change in clips. 
AudioStreamTransitioner can crossfade between different streams, or "clips". Because a transition can be activated at any time, clips will loop indefinitely until a transition is triggered. Each transition has times for fading out and fading in, based on beats. The lenght in samples of the fades is determined by the user input beats and the BPM of the two streams, meaning the fading out stream's fade in samples is calculated through its own bpm, and the fading in stream's samples are calculated through the bpm of that stream. Optionality for a transition clip also exists, which allows for a transitionary clip to be played in between the current and previous clip. The transition clip fades in while the previous fades out, plays through its samples, based either on it's length or beats, and finally fades out as the new clip fades in. 


## How to use the feature

I have made two video tutorials, explaining how to use each class, and I have also created documentation explaining what each GDScript function does, which can be found in the docs help in the editor. 

[AudioStreamPlaylist](https://youtu.be/RKMyb0WuBMc)

[AudioStreamTransitioner](https://youtu.be/1aNV5DjJu58)

## Challenges encountered 
One of the first challenges I faced was to figure out how the initial code was going to work. With some help from my mentor who explained a lot of the basics to me I managed to get a very basic version of Playlist working, however it had the problem of a memory leak, which took me a while to fix. As mentioned in the previous two progress reports, the issue was that the preview generator for audio streams was trying to create an infinite preview and that would take up all of my computer's RAM. The issue was solved through communicating with a developer interested in the feature who asked me why does Playlist loop indefinitely, at which point I realized somethig was not right. More info on this can be found in Progress report #2. 
Another big challenge was figuring out the logic for transitioner, and then adding functionalities for a transition clip in the logic. This was probably the biggest piece of mixing logic I have ever written and it was a very interesting challenge to do it properly and also debug any issues with it.
One of the problems with this was that I could hear a click when fades were happening. Figuring out the source took some back and forth in the dev chat and it was noticed that what was happening was extra frames were being processed - https://imgur.com/a/egL7Gua
I realised that the reason this was happening was because, at the end of a transition, the samples left to process would be smaller than the buffer size, which is what the mix logic uses as block size. 

    for (int i = 0; i < to_mix; i++) {
    p_buffer[i + dst_offset] = pcm_buffer[i];
    }
    dst_offset += to_mix;
    p_frames -= to_mix;
    clip_samples_total -= to_mix;
The way I fixed it was by adding a a check to see if the remaining transition samples are smaller than 0. The reason I am checking if they are smaller than 0 is that the buffer size will be subtracted from the number of remaining transition samples before the frames are sent to the final buffer. Meaning that if transition_samples < 0, that means the last block of transition samples is about to be processed. 	

    if (transition_samples<0) {
    				for (int i = 0; i < transition_samples + to_mix; i++) {
    					p_buffer[i + dst_offset] = pcm_buffer[i];
    				}
    				dst_offset += transition_samples + to_mix;
    				p_frames -= transition_samples + to_mix;
    				clip_samples_total -= transition_samples + to_mix;
    				transition_samples = 0;	
    			} else {
    				for (int i = 0; i < to_mix; i++) {
    					p_buffer[i + dst_offset] = pcm_buffer[i];
    				}
    				dst_offset += to_mix;
    				p_frames -= to_mix;
    				clip_samples_total -= to_mix;
    			}	


## What's next 
Ideally, I would like to merge the project with Godot's master branch and for it to become the go-to feature for music implementation. I plan to keep working on the two classes and try and take feedback from other users on what could be improved. I think the video tutorials I have made for this will show people how they currently work and hopefully receive useful input on how I can make them better. 
Some other things I would like to add is a counter which waits until a single beat is finished until it starts transitioning, which would make more sense than a transition starting immediately when it is triggered. I also think that it would be good to change the way crossfading works. Currently, the volume values of the fading streams are not connected to each other

    float fade_out_start_volume = 1.0 - float(fade_out_samples_total - fade_out_samples) / fade_out_samples_total;
    float fade_out_end_volume = 1.0 - float(fade_out_samples_total - (fade_out_samples - to_fade_out)) / fade_out_samples_total;						
    float fade_in_start_volume = 1.0 - float(fade_in_samples) / fade_in_samples_total;
    float fade_in_end_volume = 1.0 - float(fade_in_samples - to_fade_in) / fade_in_samples_total;		

 When fade times are different, it's possible to get the sum of their values to go over 1.0, which would cause a click. I think that finding out how to fix that will be one of the first things I will change moving forward.

## What I learned throughout GSOC and closing remarks
This project has been a very interesting experience in terms of learning how audio really works behind the scenes. Learning how the logic behind things I use almost every day works, and coming up with something on my own versions of them was very challenging and worthwhile and I now feel a lot more confident in my coding skills than I did before this year's GSOC. I also got the chance to talk to some very good developers in the dev chat who gave me some important insight into some of my problems. The complexity of this project also led me to learn how to operate a debugger, in order to find the sources of various crashes, and clicks in my crossfading logic. 
I would strongly encourage future students to participate in Google Summer of Code with Godot engine, as the community is very welcoming to all levels of experience and I have personally learned a lot! 
Thanks to this project and my last year's work I have managed to secure a few interviews with game studios so I think that for anyone looking to get that important initial experience in game engines working with Godot is an amazing opportunity! 

