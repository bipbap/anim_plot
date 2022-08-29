from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

import tkinter as tk
from tkinter import Frame,Label,Entry,Button
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.animation as animation

import argparse
import queue
import sys

from matplotlib.animation import FuncAnimation
import matplotlib.pyplot as plt
import numpy as np
import sounddevice as sd


def int_or_str(text):
    """Helper function for argument parsing."""
    try:
        return int(text)
    except ValueError:
        return text


parser = argparse.ArgumentParser(add_help=False)
parser.add_argument(
    '-l', '--list-devices', action='store_true',
    help='show list of audio devices and exit')
args, remaining = parser.parse_known_args()
if args.list_devices:
    print(sd.query_devices())
    parser.exit(0)
parser = argparse.ArgumentParser(
    description=__doc__,
    formatter_class=argparse.RawDescriptionHelpFormatter,
    parents=[parser])
parser.add_argument(
    'channels', type=int, default=[1], nargs='*', metavar='CHANNEL',
    help='input channels to plot (default: the first)')
parser.add_argument(
    '-d', '--device', type=int_or_str,
    help='input device (numeric ID or substring)')
parser.add_argument(
    '-w', '--window', type=float, default=1000, metavar='DURATION',
    help='visible time slot (default: %(default)s ms)')
parser.add_argument(
    '-i', '--interval', type=float, default=300,
    help='minimum time between plot updates (default: %(default)s ms)')
parser.add_argument(
    '-b', '--blocksize', type=int, help='block size (in samples)')
parser.add_argument(
    '-r', '--samplerate', type=float, help='sampling rate of audio device')
parser.add_argument(
    '-n', '--downsample', type=int, default=10, metavar='N',
    help='display every Nth sample (default: %(default)s)')
args = parser.parse_args(remaining)
if any(c < 1 for c in args.channels):
    parser.error('argument CHANNEL: must be >= 1')
mapping = [c - 1 for c in args.channels]  # Channel numbers start with 1
q = queue.Queue()


def audio_callback(indata, frames, time, status):
    """This is called (from a separate thread) for each audio block."""
    if status:
        print(status, file=sys.stderr)
    # Fancy indexing with mapping creates a (necessary!) copy:
    q.put(indata[::args.downsample, mapping])


def update_plot(frame):
    """This is called by matplotlib for each plot update.

    Typically, audio callbacks happen more frequently than plot updates,
    therefore the queue tends to contain multiple blocks of audio data.

    """
    global plotdata
    while True:
        try:
            data = q.get_nowait()
        except queue.Empty:
            break
        shift = len(data)
        plotdata = np.roll(plotdata, -shift, axis=0)
        plotdata[-shift:, :] = data
    for column, line in enumerate(lines):
        line.set_ydata(plotdata[:, column])
    return lines



class Window(Frame):

    def __init__(self, master = None):
        Frame.__init__(self, master)
        self.master = master
        self.init_window()


    def Clear(self):
        x=0

#    def Plot(self):
#        x=0

    def init_window(self):




        self.master.title("Use Of FuncAnimation in tkinter based GUI")
        self.pack(fill='both', expand=1)     

#Create the controls, note use of grid

        self.labelSpeed = Label(self,text="Speed (km/Hr)",width=12)
        self.labelSpeed.grid(row=0,column=1)
        self.labelAmplitude = Label(self,text="Amplitude",width=12)
        self.labelAmplitude.grid(row=0,column=2)

        self.textSpeed = Entry(self,width=12)
        self.textSpeed.grid(row=1,column=1)
        self.textAmplitude = Entry(self,width=12)
        self.textAmplitude.grid(row=1,column=2)


#        self.buttonPlot = Button(self,text="Plot",command=self.Plot,width=12)
        self.buttonPlot = Button(self,text="Plot",width=12)
        self.buttonPlot.grid(row=2,column=1)
        self.buttonClear = Button(self,text="Clear",command=self.Clear,width=12)
        self.buttonClear.grid(row=2,column=2)

#        self.buttonClear.bind(lambda e:self.Plot)
        self.buttonClear.bind(lambda e:self.Clear)



        tk.Label(self,text="SHM Simulation").grid(column=0, row=3)

        try:
            if args.samplerate is None:
                device_info = sd.query_devices(args.device, 'input')
                args.samplerate = device_info['default_samplerate']

            length = int(args.window * args.samplerate / (1000 * args.downsample))
            plotdata = np.zeros((length, len(args.channels)))

            self.fig, self.ax = plt.subplots()
            lines = self.ax.plot(plotdata)
            if len(args.channels) > 1:
                self.ax.legend([f'channel {c}' for c in args.channels],
                          loc='lower left', ncol=len(args.channels))
            self.ax.axis((0, len(plotdata), -1, 1))
            self.ax.set_yticks([0])
            self.ax.yaxis.grid(True)
            self.ax.tick_params(bottom=False, top=False, labelbottom=False,
                           right=False, left=False, labelleft=False)
            self.fig.tight_layout(pad=0)

            stream = sd.InputStream(
                device=args.device, channels=max(args.channels),
                samplerate=args.samplerate, callback=audio_callback)
            self.ani = FuncAnimation(self.fig, update_plot, interval=args.interval, blit=True)


        except Exception as e:
            parser.exit(type(e).__name__ + ': ' + str(e))
      


        self.canvas = FigureCanvasTkAgg(self.fig, master=self)
        self.canvas.get_tk_widget().grid(column=0,row=4)



root = tk.Tk()
root.geometry("800x600")
app = Window(root)
tk.mainloop()
