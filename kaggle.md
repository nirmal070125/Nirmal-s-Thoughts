Download data via command line
Ref: https://www.kaggle.com/forums/f/15/kaggle-forum/t/6604/downloading-data-via-command-line/36206

install chrome cookie plugin
export your cookies from your browser, when you logged in at kaggle and put your cookies.txt on your server. Then run:

wget -x --load-cookies cookies.txt -nH --cut-dirs=5 http://www.kaggle.com/c/dogs-vs-cats/download/test1.zip
