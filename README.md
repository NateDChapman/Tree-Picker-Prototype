# Tree-Picker-Prototype
Tree Picker Prototype Game Homework

----------------------------------------------

Apple Script

using UnityEngine;
using System.Collections;

public class Apple : MonoBehaviour {

	public static float bottomY = -20f;
	
	void Update () {
		if (transform.position.y < bottomY) {
			Destroy (this.gameObject);

			applePicker apScript = Camera.main.GetComponent<applePicker> ();
			apScript.AppleDestroyed ();
		}
	}
}

------------------------------------------------

ApplePicker Script

using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class applePicker : MonoBehaviour {

	public GameObject BasketPrefab;
	public int numBaskets =3;
	public float basketBottomY = -14f;
	public float basketSpacingY = 2f;
	public List<GameObject> basketList;

	void start () {
		basketList = new List<GameObject> ();
		for (int i=0; i<numBaskets; i++) {
			GameObject tBasketGO = Instantiate(BasketPrefab) as GameObject;
			Vector3 pos = Vector3.zero;
			pos.y = basketBottomY = (basketSpacingY * i);
			tBasketGO.transform.position = pos;
			basketList.Add (tBasketGO);
		}
	}

	public void AppleDestroyed() {
		GameObject[] tAppleArray = GameObject.FindGameObjectsWithTag ("Apple");
		foreach (GameObject tGO in tAppleArray) {
			Destroy (tGO);
		}

		int basketIndex = basketList.Count - 1;
		GameObject tBasketGO = basketList [basketIndex];
		basketList.RemoveAt (basketIndex);
		Destroy (tBasketGO);

		if (basketList.Count ==0) {
			Application.LoadLevel("Scene_0");
		}
	}
}

-------------------------------------

Apple Tree Script

using UnityEngine;
using System.Collections;

public class appleTree : MonoBehaviour {

	public GameObject ApplePrefab;
	public float speed = 1f;
	public float leftAndRightEdge = 10f;
	public float chanceToChangeDirections = 0.1f;
	public float secondsBetweenAppleDrops = 1f;

	void Start () {
		InvokeRepeating ("DropApple", 2f, secondsBetweenAppleDrops);
	}

	void DropApple() {
		GameObject Apple = Instantiate (ApplePrefab) as GameObject;
		Apple.transform.position = transform.position;
	}

	void Update () {
		Vector3 pos = transform.position;
		pos.x += speed * Time.deltaTime;
		transform.position = pos;
		if (pos.x < -leftAndRightEdge) {
			speed = Mathf.Abs (speed);
		} else if (pos.x > leftAndRightEdge) {
			speed = -Mathf.Abs (speed);
		}
	}
	void FixedUpdate() {
			if (Random.value < chanceToChangeDirections) {
				speed *= -1; // CHANGE DIRECTION
			}
	}
}

----------------------------------------

Basket Script

using UnityEngine;
using System.Collections;

public class Basket : MonoBehaviour {

	public GUIText scoreGT;

	void Start() {
		GameObject scoreGO = GameObject.Find ("ScoreCounter");
		scoreGT = scoreGO.GetComponent<GUIText> ();
		scoreGT.text = "0";
	}
	void Update () {
		Vector3 mousePos2D = Input.mousePosition;
		mousePos2D.z = -Camera.main.transform.position.z;
		Vector3 mousePos3D = Camera.main.ScreenToWorldPoint (mousePos2D);
		Vector3 pos = this.transform.position;
		pos.x = mousePos3D.x;
		this.transform.position = pos;
	}
	void OnCollissionEnter(Collision Coll) {
		GameObject collidedWith = Coll.gameObject;
		if (collidedWith.tag == "Apple") {
			Destroy (collidedWith);
		}

		int score = int.Parse (scoreGT.text);
		score += 100;
		scoreGT.text = score.ToString ();

		if (score > HighScore.score) {
			HighScore.score = score;
		}
	}
}

---------------------------------------------

High Score Script

using UnityEngine;
using System.Collections;

public class HighScore : MonoBehaviour {
	static public int score = 1000;

	void Awake() {
		if (PlayerPrefs.HasKey ("ApplePickerHighScore")) {
			score = PlayerPrefs.GetInt ("ApplePickerHighScore");
		}
	}

	// Update is called once per frame
	void Update () {
		GUIText gt = this.GetComponent<GUIText> ();
		gt.text = "High Score: " + score;

		if (score > PlayerPrefs.GetInt ("ApplePickerHighScore")) {
			PlayerPrefs.SetInt ("ApplePickerHighScore", score);
		}
	}
}
