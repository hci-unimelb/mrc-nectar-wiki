# Instructions

We first need to open the ports.
So go in your MRC Dashboard, then from the left sidebar go to `network -> Security groups`.
I set it my Jupyter notebook to run on port `7000`.
Add the following rules for that port:
* TCP Ingress
* TCP Egress
* UDP Ingress
* UDP Egress

Jupyter also uses `websockets` so add another set of those rules (`TCP` Ingress/Egress and `UDP` Ingress/Egress) with port range `49152-65535`.

After this, go to Compute -> Instances, from your instance click on the little dropdown on the right, under Actions and select "Edit Security Groups". Move all the security groups you need to the right

Generate the config file with:
```bash
$ jupyter notebook --generate-config
```
And then edit it with:
```bash
$ sudo nano ~/.jupyter/jupyter_notebook_config.py
```

Anyways once you get that working, generate a jupyter notebook config file and uncomment and change these lines:
```bash
c.NotebookApp.allow_origin = '*'
c.NotebookApp.allow_remote_access = True
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.notebook_dir = '/var/www/external/gabry/project/' # or whatever folder you prefer
c.NotebookApp.open_browser = False
c.NotebookApp.port = 7000
#  The string should be of the form type:salt:hashed-password.
c.NotebookApp.password = u'sha1:ae1c52ce1e8a:2f010047f883c5950a76acbe8a977c057a5e49db'
```
It took me ages to find these lines to add in the config file, without these it does not work.

Last tip, add these lines at the end of your `.bashrc`. So `sudo nano ~/.bashrc` and then add these:
```bash
$ alias start_jupyter_notebook='sudo nohup jupyter notebook --allow-root &'
$ alias start_jupyter_lab='sudo nohup jupyter lab --allow-root --no-browser &'
$ alias sudo='sudo '
```
Then run this (do not `sudo` this one):
```bash
source ~/.bashrc
```

So now you can start either of them in the background by just calling one of these two So you won't need to keep the ssh open
```bash
$ start_jupyter_notebook
(or) $ start_jupyter_lab
```

If by any chance something is wrong and you need to restart the notebook or lab just call:
```bash
$ ps -ef | grep jupyter
```
Look for the PID and then just call:
```bash
$ sudo kill [PID]
```

#  Extensions
## Install jupyterthemes
```bash
$ conda install -c conda-forge jupyterthemes

$ jupyter nbextension install https://rawgithub.com/minrk/ipython_extensions/master/nbextensions/gist.js

$ jupyter nbextension enable gist

$ jupyter nbextension install --user https://rawgithub.com/minrk/ipython_extensions/master/nbextensions/toc.js

$ curl -L https://rawgithub.com/minrk/ipython_extensions/master/nbextensions/toc.css > $(jupyter --data-dir)/nbextensions/toc.css
```

Alternatively just run this and check the folder path:
```bash
jupyter --data-dir
```
 
Download toc.ss from [here](https://rawgithub.com/minrk/ipython_extensions/master/nbextensions/toc.css) and put it in: `C:\Users\Gabryxx7\AppData\Roaming\jupyter\nbextensions`
```bash
$ jupyter nbextension enable toc
```

Update to latest version
```bash
$ conda update jupyterthemes
```

## Custom themes
Theme customised by [Gabryxx7](https://github.com/Gabryxx7)
```bash
jt -t onedork -fs 10 -altp -tfs 135 -nfs 115 -cellw 80% -T -f firacode
```

`jt_toc.css`:
```CSS
/* extracted from https://gist.github.com/magican/5574556
   slightly modified for the selectors of toc-items */

#toc {
  overflow-y: auto;
  max-height: 300px;
  padding: 0px;
}

#toc ol.toc-item {
    counter-reset: item;
    list-style: none;
    padding: 0.1em;
}

#toc ol.toc-item  li {
    display: block;
}

#toc ol.toc-item li:before {
    counter-increment: item;
    content: counters(item, ".")" ";
}

#toc-wrapper {
  position: fixed;
  top: 120px;
  max-width:230px;
  left: 30px;
  border: thin solid rgba(0, 0, 0, 0.38);
  border-radius: 5px;
  padding:10px;
  background-color: #fff;
  opacity: .8;
  z-index: 100;
}

#toc-wrapper.closed {
  min-width: 100px;
  width: auto;
  transition: width;
}
#toc-wrapper:hover{
  opacity: 1;
}
#toc-wrapper .header {
  font-size: 18px;
  font-weight: bold;
}
#toc-wrapper .hide-btn {
  font-size: 14px;
  font-family: monospace;
}

#toc-wrapper .reload-btn {
  font-size: 14px;
  font-family: monospace;
}


/* don't waste so much screen space... */
#toc-wrapper .toc-item{
  padding-left: 20px;
}

#toc-wrapper .toc-item .toc-item{
  padding-left: 10px;
}
```