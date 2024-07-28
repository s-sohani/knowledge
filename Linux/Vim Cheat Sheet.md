:w ذخیره‌ی تغییرات  
:wq / :x ذخیره‌ی تغییرات و خروج  
:q خروج  
:q! خروج بدون ذخیره‌ی تغییرات ذخیره نشده  
Exiting insert mode  
Esc خروج از حالت درج  
Visual mode  
v ورود به حالت visual  
V ورود به حالت visual line  
In visual mode  
d / x پاک کردن موارد انتخاب شده  
y کپی موارد انتخاب شده  
Navigating  
gg رفتن به ابتدای فایل  
G رفتن به آخر فایل  
:n رفتن به خط n ام  
n رفتن به مورد بعدی یافته شده در جستجو  
N رفتن به مورد قبلی یافته شده در جستجو  
* اشاره گر به محل بعدی آن لغتی که الان روش هست می‌ره  
Editing  
i وارد شدن به حالت درج  
R وارد شدن در حالت جایگزینی  
u برگرداندن تغییرات  
Ctrl + r عملگر redo  
Clipboard  
x پاک کردن کاراکتری که روش هستی  
dd برش خط (همون cut خارجکی‌ها)  
yy کپی کردن خط  
p الصاق اونی که کپی شده  
Others  
/word جستجو  
:set hls اگه این کار رو بکنی همه‌ی یافته‌های توی search رنگی می‌شن  
:set number نمایش شماره‌ی خط  
:set number! لغو نمایش شماره‌ی خط  
:!command اجرای یک دستور در شل  
:%s/old/new/g جایگزینی عبارت old با new در سراسر فایل


___


Vim (Vi IMproved) is a highly configurable text editor built to make creating and changing any kind of text very efficient. It is an enhanced version of the vi editor distributed with most UNIX systems. Vim is often called a "programmer's editor," and so useful for programming that many consider it an entire IDE. It's not just for programmers, though. Vim is perfect for all kinds of text editing, from composing email to editing configuration files.

	Normal mode For navigation and text manipulation.
	Insert mode For inserting text.
	Visual mode For selecting text.
	Command-line mode For entering commands.
	i Enter insert mode to start editing text.
	R Enter replace mode
	Esc Return to normal mode from insert mode.
	w Save changes.
	q Quit Vim.
	wq Save changes and quit Vim.
	q! Quit without saving changes.
	v Enter visual mode
	V Enter visual line mode
	d / x Delete selected items
	y Copy selected items
	gg Go to the beginning of the file
	G Go to the end of the file
	:n Go to line number n
	n Go to the next search result
	N Go to the previous search result Move the cursor to the next occurrence of the word under the cursor
	x Delete the character under the cursor
	dd Cut the current line
	yy Copy the current line
	p Paste
	u Undo changes
	Ctrl + r Redo
	/word Search for "word"
	:set hls Highlight all search results
	:set number Show line numbers
	:set number! Hide line numbers
	:!command Execute a shell command
	:%s/old/new/g Replace "old" with "new" throughout the file