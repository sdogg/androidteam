几个实验，大家选一个做，争取在一周内搞定。

1，屏幕下方两个button：button1，button2（名字可以随便起），上方是textview或者别的你能用的。点击button1 时textview显示button1，点击button2时textview显示button2。

2，屏幕下方两个button：button1，button2（名字可以随便起），上方时textview。但是条件是初始时只显示button1，当点击button1时有如下操作：button1消失，button2显示出来，且textview显示“you clicked button1”.当点击button2时有如下操作：button2消失，button1显示出来，且textview显示“you clicked button2”。

3，之前做的汉字下落的功能用的是TranslateAnimation。但是如果此动画结束后在屏幕上会显示出来原先的textview的内容。现在的要求如下：初始时屏幕上只有下方有个button，上方为黑屏，点击button，开始执行此动画，请用setRepeatCount(1000)以使此动画能够在较长时间内一直执行，然后有另一个button名为button2，点击此button后，该动画无论在何状态，即无论该字落到何处，立即使该动画消失，恢复原先黑屏状态。

PS：第三题或许较难，请大家都考虑下如何实现。