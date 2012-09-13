
build:
	virtualenv --no-site-packages .
	bin/pip install landslide

render:
	bin/landslide circus.cfg
	bin/landslide rpi.cfg
	bin/landslide marteau.cfg

