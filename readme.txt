# 
# some experiments with ffmpeg
# 2020-11-01
#

#
# cut out a section
#
# first 30 sec
# ffmpeg -i original.mp4 -t 30 -c copy -map 0 snip.mp4
#
# from t=0 to t=5 sec
# ffmpeg -i snip.mp4 -ss 00:00 -t 00:05  snip_5sec.mp4 
#

#
# scale and crop
#
# ffmpeg -i snip.mp4 -vf 'scale=2.4*iw:-1, crop=iw/2.4:ih/2.4:400:600' cropped.mp4
#

#
# slow it down
#
# ffmpeg -i snip5.mp4 -filter_complex "[0:v]setpts=2.0*PTS[v];[0:a]atempo=0.5[a]" -map "[v]" -map "[a]"  snip5_slo.mp4
#

#
# insert keyframes
#
# ffmpeg -i timestamp_3-zoom_3.mp4 -force_key_frames 'expr:gte(t,n_forced)' timestamp_3-zoom_3-kf.mp4 
#

#
# overlay timecode 
#
# ffmpeg -i snip_5sec.mp4 -vf "drawtext=text='%{pts \: flt}':box=1:fontsize=24:x=10:y=20" snip_5sec_box.mp4
#

#
# play 1.2 seconds starting 2 sec in and loop forever
#
# ffplay -ss 2 -t 1.2 -loop 0 -volume -30 snip5_slo.mp4
#

#
# count iframes
#
# ffprobe -select_streams v -show_frames snip.mp4 | grep key_frame=1 | wc -l
#

#
# find duration
#
# ffprobe  2>&1 original.mp4 |  grep Duration
#

#
# extract keyframes into pts-named image files
#
# ffmpeg -i snip.mp4  -vf "select='eq(pict_type, PICT_TYPE_I)'" -vsync vfr -frame_pts true images/pts%03d.png 
# 
# ?
# ffmpeg -i snip.mp4 -vf "select='eq(pict_type, PICT_TYPE_I)'" -vsync vfr images/iframe%03d.png
#

#
# translate pts filenames in frames to seconds or 10*seconds 
#
## should take dir, tenths flags as args
#
zmodload zsh/mathfunc
function translate_filenames() {	

	for f in images/pts*; do
		g=`basename $f`; q=$(z=${g##pts}; print ${z%\.*});

		# in tenths of a second
		(( w = ((q / 29.97)*10) ));

		# in seconds
		# (( w = ((q / 29.97)) ));

		echo "w: " $(( (w) ));

		int_part=$(( int(w)/10 ));  
		echo $int_part
		
		frc_part=$(( int( (w % 1.0)*10) ));  
		echo $frc_part

		echo xmv $f `dirname $f`'/pts'$int_part'_'$frc_part'/png'
	done
}

