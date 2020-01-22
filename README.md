# 各專案精選程式碼彙整
此處節錄介紹我在各個專案，包刮編輯器《DialogueTree》、遊戲專案《沉沒意志》、遊戲專案《圖塔圖塔》中較為精華的程式碼片段

## 圖形化對話編輯器DialogueTree

#### DialogueTree類別
負責編輯器視窗的實際繪製，以及調用各個面版類別的點擊與資料設定/取得函式
```cs
public class DialogueTree : EditorWindow {

//根據編輯器的不同狀態，ProcessEvent函式可區分出當前需進行的事件
public enum WindowState{normal, drag, move, popup, link, scroll};
public WindowState nowState = WindowState.normal;

public List<Character> lst_chars = new List<Character> ();	//角色清單
public List<Node> lst_node = new List<Node> ();			//節點清單
Vector2 coordinate;						//畫面座標
```

OnGUI負責主視窗與各面板的繪製，同時也從這邊取得使用者的輸入並做相應的處理
```cs
void OnGUI(){
	//根據編輯器的不同操作狀態，偵測使用者的輸入(點擊、拖曳等)來做出相應的處理
	ProcessEvent (Event.current);
	
	DrawBackground ();		//繪製背景網格
	DrawNodes ();			//以迴圈跑過節點清單中的每個節點，呼叫其繪製函式
	DrawPanels ();			//繪製浮動在主畫面之上的各個面板
	Repaint ();
}
```

#### Node類別
基本節點類別，其餘各種節點類別(StartNode、DialogueNode等等)皆繼承自此類別
```cs
public class Node {

public Rect rect;		//記錄此節點的原始Rect
public Rect canvasRect;		//由於主視窗的畫面可隨意移動，因此設立此變數來記錄實際在畫面中顯示的Rect
```
繪製節點函式
```cs
//繪製函式設為virtual，以便其他種類的節點在繼承為子類別後，可以依據各自不同的資訊來覆寫更改繪製函式
virtual public void DrawSelf(Vector2 coordinate){
	//將此節點的原始Rect經過主視窗的畫面座標換算後得到新的Rect
	canvasRect = rect;
	canvasRect.position += coordinate;
	
	//繪製節點上的各個元素
	GUI.BeginGroup(canvasRect);
	if (isSelected)
		GUI.DrawTexture(outlineRect, tex_selected);
	GUI.DrawTexture(boxRect, tex_normal);
	GUI.Label (NameRect, nodeName, style_name);
	GUI.EndGroup ();
}
```
#### MyFunctions類別
匯集各通用函式，方便從各個類別中調用
```cs
public class MyFunctions {
```
將各個節點吸附至背景等間隔的格點上，方便節點的標齊對正
```cs
public static Vector2 SnapPos(Vector2 size, Vector2 position, float range){
	Vector2 center = position + 0.5f * size;
	center.x = Mathf.Round (center.x / range) * range;
	center.y = Mathf.Round (center.y / range) * range;
	return center - 0.5f * size;
}
```
使用者輸入空字串時會自動修正為value參數的字串，避免出現空字串。
```cs
public static string ClampString(string target, string value){
	char[] splitter = new char[]{ ' ' };
	return (target.Split (splitter, System.StringSplitOptions.RemoveEmptyEntries).Length == 0) ? value : target;
}
```
將_name字串賦予給target，成為其角色名
為避免角色名重複，當_name參數字串與角色名單中的任一角色名重複時，會在_name字串後端增加數字，再賦予給target
```cs
public static string SetName(string _name, Character target, List<Character> charList){
	foreach(Character c in charList) {
	    if (_name == c.name && c != target) {
		string[] sepName = _name.Split (new char[]{ ' ' }, System.StringSplitOptions.RemoveEmptyEntries);
		int number = 0;
		if (int.TryParse (sepName [sepName.Length - 1], out number)) {
		    string result = "";
		    for (int i = 0; i < sepName.Length - 1; i++)
			result += sepName [i] + " ";
		    return SetName (result + (number + 1).ToString (), target, charList);
		} else
		    return SetName (_name + " 1", target, charList);
	    }
	}
	return _name;
}
```
## 遊戲專案《沉沒意志》
#### 遊戲區域
程式要能夠偵測玩家進入哪一個遊戲區域，藉此觸發各種不同事件。\
由於"玩家進入X區域"此一訊息需要傳達給各種不同的類別，在這裡使用了觀察者模式來實作此功能。

建立區域觀察者介面
```cs
public interface i_AreaObserver{ void ChangeArea (string _nowArea); }
```
在每個區域的遊戲物件上都掛載此sc_Area腳本，依據各自的判定範圍偵測玩家是否進入自身區域
```cs
public class sc_Area : MonoBehaviour {
	[SerializeField]
	string AreaCode = "A0";			//每個實例區域擁有自己的區域代號
	static string NowArea = "";		//以靜態變數紀錄玩家現在所處的區域代號，讓各個實例區域都能共享此資訊
	static List<i_AreaObserver> myObservers = new List<i_AreaObserver> ();	//以靜態變數共享觀察者名單
	
	//加入觀察者
	static public void RegisterObserver(i_AreaObserver _observer){ myObservers.Add (_observer); }
	
	if (AreaCode != NowArea) {		//若當前紀錄的玩家所在區域不等於自身區域時進行偵測
		if (CheckPlayerPosition()) {	//偵測玩家的實際座標是否在此區域內
			NowArea = AreaCode;
			//呼叫每個觀察者的改變區域函式，並將當前區域代號作為參數傳入
			foreach (i_AreaObserver AO in myObservers)
				AO.ChangeArea (NowArea);
		} 
	}
```
繼承了區域觀察者介面的類別就可依此觸發不同事件

#### 劇情轉折點
由對話中的玩家所選的選項或是與場景上的物件互動皆可增加或移除轉折點。\
劇情轉折點也運用了觀察者模式，加入轉折點或移除轉折點都可讓不同觀察者觸發不同事件，在此就不多贅述。
```cs
public interface i_PlotFlag{
	void FlagAdd (string _key);
	void FlagRemove (string _key);
}
```

## 遊戲專案《圖塔圖塔》
遊戲由雙方隊伍派兵互相進攻，在同一條道路上的友兵會排隊增加「重量」，與敵兵碰撞時會以此重量決定碰撞結果(往重量輕的一方推進或僵持不下)。\
其排隊與碰撞系統皆在基礎士兵腳本中實作。
#### 士兵腳本
```cs
public class sc_Hero : MonoBehaviour {
	//記錄各個基礎素質與前後連結的士兵
	[SerializeField]
	float Weight;				//自身本體的重量，此數值在遊戲中不會變動
	public float totalWeight;		//記錄隊伍總重，此數值會在runtime根據排隊情況作加減
	public sc_Hero FrontConnect = null;	//排隊時，記錄前方士兵
	public sc_Hero BackConnect = null;	//排隊時，記錄後方士兵
	
	//每幀會依序進行下列判斷
	void Update () {
		//若往上推進到敵方主堡，或被往下回推到己方主堡，或是血量歸0，則消滅此士兵
		if (transform.position.y < -4.5f || transform.position.y > 4.5f || hp < 0.01f)
			SelfDestruct();

		CheckConnect ();			//偵測與友軍的連結(排隊系統)
		if (face == 1)	CheckCollide();		//偵測與敵軍的碰撞(碰撞系統)，由其中一側的士兵來進行此判斷即可
		Move ();				//依據碰撞後的結果進行移動
	}
```
#### 排隊系統
每個士兵皆會執行偵測友軍連結的函式，與後方友軍連結時會將此友軍所記錄的總重量加在自身總重量上，如此一來排成一排的士兵將會形成遞迴關係，將重量從最後方一路加總到最前方的士兵身上。
```cs
void CheckConnect(){
	//向後偵測友軍距離
	RaycastHit2D other = Physics2D.Raycast(transform.position, face * Vector2.down, range, layerMask);
	if (other.collider != null) {
		//若後方友軍距離夠近，則排成隊伍並將後方友軍所記錄的總重增加到自身總重上
		if (BackConnect == null) {
			BackConnect = other.collider.GetComponent<sc_Hero>();
			BackConnect.FrontConnect = GetComponent<sc_Hero>();
		}
		totalWeight = Weight + BackConnect.totalWeight;
	}else{
		//若不夠近則切斷隊伍，總重等同於自己本體的重量
		if (BackConnect != null) {
			BackConnect.FrontConnect = null;
			BackConnect = null;
		}
		totalWeight = Weight;
	}
}
```
#### 碰撞系統
敵我雙方的最前方士兵都經由排隊系統得到整排隊伍的總重量後，再互相進行碰撞判斷。
```cs
void CheckCollide(){
	//向前偵測是否碰撞到敵方士兵
	RaycastHit2D other = Physics2D.Raycast(transform.position, Vector2.up, range, layerMask);
	if (other.collider != null) {
		sc_Hero enemy = other.collider.GetComponent<sc_Hero> ();
		//雙方互相受到攻擊傷害
		enemy.Damaged (ATK);
		Damaged (enemy.ATK);
		
		//以雙方隊伍的總重量判斷碰撞後往哪一方推進，或是等重而相互後退
		float otherWeight = enemy.totalWeight;
		if (totalWeight > otherWeight + 0.01f) {
			enemy.ConnectSpeed(backSPD, true);
			spd = 0f;
			Anim.SetTrigger ("attack");
		} else if (totalWeight < otherWeight - 0.01f) {
			enemy.ConnectSpeed(0f, true);
			spd = backSPD;
			enemy.Anim.SetTrigger ("attack");
		} else {
			enemy.ConnectSpeed(backSPD, true);
			spd = backSPD;
		}
	}
}
```

