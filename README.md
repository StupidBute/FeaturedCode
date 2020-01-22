# 各專案的精選程式碼彙整
此處介紹我在各個專案，包刮編輯器《DialogueTree》、遊戲專案《沉沒意志》、遊戲專案《圖塔圖塔》中較為精華的程式碼片段

## 圖形化對話編輯器DialogueTree

#### DialogueTree類別
	public class DialogueTree : EditorWindow
負責編輯器視窗的實際繪製，以及調用各個面版類別的點擊與資料設定/取得函式

	void OnGUI(){
	DrawBackground ();
	ProcessEvent (Event.current);
	DrawNodes ();
	DrawPanels ();
	Repaint ();
	}
OnGUI負責主視窗與各面板的繪製，同時也從這邊取得使用者的輸入並做相應的處理

#### Node類別
	public class Node
基本節點類別，其餘各種節點類別(StartNode、DialogueNode等等)皆繼承自此類別

	public class Node

	public Rect rect;		//記錄此節點的原始Rect
	public Rect canvasRect;		//由於主視窗的畫面可隨意移動，因此設立此變數來記錄實際在畫面中顯示的Rect
	
	//繪製節點函式
	virtual public void DrawSelf(Vector2 coordinate){
		//將此節點的原始Rect經過主視窗的畫面座標換算後得到新的Rect
		canvasRect = rect;
		canvasRect.position += coordinate;

		GUI.BeginGroup(canvasRect);
		if (isSelected)
			GUI.DrawTexture(outlineRect, tex_selected);
		GUI.DrawTexture(boxRect, tex_normal);
		GUI.Label (NameRect, nodeName, style_name);
		GUI.EndGroup ();
	}
	
#### MyFunctions類別
	public class MyFunctions
匯集各通用函式，方便從各個類別中調用

	//將各個節點吸附至背景等間隔的格點上，方便節點的標齊對正
	public static Vector2 SnapPos(Vector2 size, Vector2 position, float range){
	Vector2 center = position + 0.5f * size;
	center.x = Mathf.Round (center.x / range) * range;
	center.y = Mathf.Round (center.y / range) * range;
	return center - 0.5f * size;
	}

	//使用者輸入空字串時會自動修正為value參數的字串，避免出現空字串。
	public static string ClampString(string target, string value){
	char[] splitter = new char[]{ ' ' };
	return (target.Split (splitter, System.StringSplitOptions.RemoveEmptyEntries).Length == 0) ? value : target;
	}

	//將_name字串賦予給target，成為其角色名
	//為避免角色名重複，當_name參數字串與角色名單中的任一角色名重複時，會在_name字串後端增加數字，再賦予給target
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

## 遊戲專案《沉沒意志》
### 

## 遊戲專案《圖塔圖塔》
### 
