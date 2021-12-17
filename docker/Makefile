run:
	docker run -it --gpus all -v /tmp/.X11-unix:/tmp/.X11-unix -v ${PWD}/../scenes:/ADOP/scenes -v ${PWD}/../experiments:/ADOP/experiments -e DISPLAY=$DISPLAY -e QT_X11_NO_MITSHM=1 --cpus 10 -m 48g adop:latest bash
