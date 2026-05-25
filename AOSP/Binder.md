## structure
>为了IPC通信

<img width="1470" height="829" alt="image" src="https://github.com/user-attachments/assets/2a003463-e117-4c6a-b213-a52e1901cc26" />

## case

### no IPC
<img width="1480" height="657" alt="image" src="https://github.com/user-attachments/assets/c801be9a-919b-4509-ad53-34da5dedede0" />

这里特殊的地方是**没有**IPC通信，完全只是为了遵循AOSP IPC的设计。**因为Bp和Bn都属于libmedia**。

### IPC

<img width="981" height="592" alt="image" src="https://github.com/user-attachments/assets/acc0fbf0-dc1c-4327-9686-0cdfd016c34e" />

这是遵循AOSP IPC的设计并**有**IPC通信的情况。

**Bp属于libmedia，Bn属于libmediaplayerservice**。
