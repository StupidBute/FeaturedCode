# 各專案的精選程式碼彙整

## 圖形化對話編輯器DialogueTree
### DialogueTree類別
負責編輯器視窗的實際繪製，以及調用各個面版類別的點擊與資料設定/取得函式
    
    public class DialogueTree : EditorWindow
    
    //OnGUI負責主視窗與各面板的繪製，同時也從這邊取得使用者的輸入並做相應的處理
    void OnGUI(){
      DrawBackground ();
      ProcessEvent (Event.current);
      DrawNodes ();
      DrawPanels ();
      Repaint ();
    }

### Node類別
基本節點類別，其餘各種節點類別(StartNode、DialogueNode等等)皆繼承自此類別

    public class Node
    
    #region Draw
	virtual public void DrawSelf(Vector2 coordinate){
		canvasRect = rect;
		canvasRect.position += coordinate;
		GUI.BeginGroup(canvasRect);
		if (isSelected)
			GUI.DrawTexture(outlineRect, tex_selected);
		GUI.DrawTexture(boxRect, tex_normal);
		GUI.Label (NameRect, nodeName, style_name);
		GUI.EndGroup ();
	}

	virtual public void DrawMyLink(){
		foreach (NodeLink nl in NextLink)
			nl.DrawSelf ();
  	}
    #endregion

    #region Hit
	virtual public Node HitTest(Vector2 mousePos){
		if (canvasRect.Contains (mousePos)) {
			Selected (true);
			EditorGUI.FocusTextInControl ("");
			return this;
		} else
			return null;
	}

	virtual public bool HitLinkTest(Vector2 mousePos){
		if (NextLink.Count > 0 && NextLink[0].HitTest (mousePos))
			return true;
		return false;
	}
    #endregion


### MyFunctions類別
匯集各通用函式，方便從各個類別中調用

    public class MyFunctions
    
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
