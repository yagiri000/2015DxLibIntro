#DxLib導入講座：課題解答

##マウスの位置に画像を描画  
DrawGraph関数を使って描画してもよいのですが、DrawRotaGraph関数を使って描画すると、画像の中央の座標を指定して描画できるので便利です。

```cpp
#include <DxLib.h>
#include <stdio.h>


int mousex = 0;//マウス座標
int mousey = 0;

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{

	ChangeWindowMode(TRUE);//非全画面にセット
	SetGraphMode(640, 480, 32);//画面サイズ指定
	SetOutApplicationLogValidFlag(FALSE);//Log.txtを生成しないように設定
	if (DxLib_Init() == 1){ return -1; }//初期化に失敗時にエラーを吐かせて終了

	int handle = LoadGraph("sample.png");

	while (ProcessMessage() == 0)
	{
		ClearDrawScreen();//裏画面消す
		SetDrawScreen(DX_SCREEN_BACK);//描画先を裏画面に

		GetMousePoint(&mousex, &mousey); //マウス座標更新
				
		DrawRotaGraph(mousex, mousey, 1.0, 0.0, handle, 0);

		ScreenFlip();//裏画面を表画面にコピー
	}

	DxLib_End();
	return 0;
}
```

##上下左右キーで対応した方向に画像を動かす  
* x,yという変数を用意しておいて、それに値を足したり引いたりすることで、上下左右に画像を動かすことができます。  
* x,yをint型にすると、一フレームあたり0.5pxずつ動かすという処理がかけないので、実数を扱えるdouble型を使いました。  
* 速度を表す変数speedを用意しました。if文の中に'''x += 2.5'''とそのまま書いても動くことは動くのですが、そうすると速度を変えたいと思った時に4箇所書き換えなければいけないので手間がかかります。また、speedという変数を使うことで、2.5が速度であることをあとで読んだ時わかりやすくなります。変数名も、ある意味コメントのようなものなので、プログラムに慣れてきたら考えてつけたほうがいいです。  

```cpp
#include <DxLib.h>
#include <stdio.h>

//キー取得用の配列
char buf[256] = { 0 };
int keystate[256] = { 0 };


//キー入力状態を更新する関数
void keyupdate()
{
	GetHitKeyStateAll(buf);
	for (int i = 0; i< 256; i++)
	{
		if (buf[i] == 1) keystate[i]++;
		else keystate[i] = 0;
	}
}

int handle;
double x = 320;//座標
double y = 240;
double speed = 2.5;//速度

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{

	ChangeWindowMode(TRUE);//非全画面にセット
	SetGraphMode(640, 480, 32);//画面サイズ指定
	SetOutApplicationLogValidFlag(FALSE);//Log.txtを生成しないように設定
	if (DxLib_Init() == 1){ return -1; }//初期化に失敗時にエラーを吐かせて終了

	handle = LoadGraph("sample.png");

	while (ProcessMessage() == 0)
	{
		ClearDrawScreen();//裏画面消す
		SetDrawScreen(DX_SCREEN_BACK);//描画先を裏画面に

		keyupdate();//キー入力状態を更新する（自作関数）

		if (keystate[KEY_INPUT_RIGHT] > 0){
			x += speed;
		}
		if (keystate[KEY_INPUT_LEFT] > 0){
			x += -speed;
		}
		if (keystate[KEY_INPUT_DOWN] > 0){
			y += speed;
		}
		if (keystate[KEY_INPUT_UP] > 0){
			y += -speed;
		}

		DrawRotaGraph(x, y, 1.0, 0.0, handle, 0);

		ScreenFlip();//裏画面を表画面にコピー
	}

	DxLib_End();
	return 0;
}
```

##EX演習問題
* 画像に(320, 240)を中心とする、円運動をさせよ。  
円運動を行うには、sin,cos関数を使うとよい。sin,cos関数を使うには、math.hをインクルードする。  

* 原点(0,0)からマウスの座標に緑の線を引け。また、原点からマウスへの角度を表示せよ。  
線を引くにはDrawLine関数、文字を表示するにはDrawFormatString関数、角度を取得するにはmath.hのatan2関数を使う。
>atan2関数について  
>atan2関数は、原点(0,0)から、(x,y)とx軸がなす角を求め、ラジアンで返す関数です。  
>引数の順番が、y,xであることに注意してください。  

```atan2(y, x)```


##EX演習問題解答

##画像に(320, 240)を中心とする、円運動をさせよ  
* sin,cos関数を使うと、円運動を行うことができます。sin,cos関数を使うには、math.hをインクルードします。  
* そのために何フレーム経過したか測定するための変数、countを用意し、それに応じて回転角を増加させました。  

```cpp
#include <DxLib.h>
#include <stdio.h>
#include <math.h>

int handle;
int count = 0;

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{

	ChangeWindowMode(TRUE);//非全画面にセット
	SetGraphMode(640, 480, 32);//画面サイズ指定
	SetOutApplicationLogValidFlag(FALSE);//Log.txtを生成しないように設定
	if (DxLib_Init() == 1){ return -1; }//初期化に失敗時にエラーを吐かせて終了

	handle = LoadGraph("sample.png");

	while (ProcessMessage() == 0)
	{
		//ClearDrawScreen();//裏画面消す
		SetDrawScreen(DX_SCREEN_BACK);//描画先を裏画面に

		count++;

		double r = 100;
		double angle = count * 0.1;
		double x = r * cos(angle) + 320;
		double y = r * sin(angle) + 240;

		DrawRotaGraph(x, y, 1.0, 0.0, handle, 0);

		ScreenFlip();//裏画面を表画面にコピー
	}

	DxLib_End();
	return 0;
}
```


##原点(0,0)からマウスの座標に緑の線を引け。また、原点からマウスへの角度を表示せよ。  
* まず、このような問題が出て、どうすればいいかよくわからない場合、一気にやろうとするのではなく、一つ一つやっていきましょう。具体的に言えば、「緑の線を引く」と「角度表示」を一度にやるのではなく、「緑の線を引く」ができてから、「角度を表示」を行うコードを書き始める、ということです。  

* DrawFormatString関数は、printf関数と同じで、int型を表示するには%d, double型を表示するには%fを使います。  

>atan2関数について  
>atan2関数は、原点(0,0)から、(x,y)とx軸がなす角を求め、ラジアンで返す関数です。
>帰ってくる値angleは、　-PI < angle < PIです。(PI = 3.14159...)  
>引数の順番が、y,xであることに注意してください。  

```cpp
atan2(y, x)
```


```cpp  
#include <DxLib.h>
#include <stdio.h>
#include <math.h>

int mousex = 0;//マウス座標
int mousey = 0;


int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
	ChangeWindowMode(TRUE);//非全画面にセット
	SetGraphMode(640, 480, 32);//画面サイズ指定
	SetOutApplicationLogValidFlag(FALSE);//Log.txtを生成しないように設定
	if (DxLib_Init() == 1){ return -1; }//初期化に失敗時にエラーを吐かせて終了

	while (ProcessMessage() == 0)
	{
		ClearDrawScreen();//裏画面消す
		SetDrawScreen(DX_SCREEN_BACK);//描画先を裏画面に

		GetMousePoint(&mousex, &mousey); //マウス座標更新

		double angle = atan2((double)mousey, (double)mousex);

		//線を描画
		DrawLine(0, 0, mousex, mousey, GetColor(0, 255, 0));

		//左上にmousex,mousey,angleを描画
		DrawFormatString(20, 40, GetColor(255, 255, 255), "MX:%3d MY:%3d", mousex, mousey);
		DrawFormatString(20, 60, GetColor(255, 255, 255), "angle:%f", angle);

		ScreenFlip();//裏画面を表画面にコピー
	}

	DxLib_End();
	return 0;
}
```