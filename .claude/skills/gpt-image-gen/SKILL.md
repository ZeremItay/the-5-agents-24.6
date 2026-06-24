---
name: gpt-image-gen
description: מעטפת לקריאת OpenAI Images API ליצירת תמונה מ-prompt. שולח prompt ומקבל בחזרה קובץ PNG. משמש את יובל (מעצב התמונות) ליצירת כל התמונות בפרויקט.
---

# gpt-image-gen — יצירת תמונות דרך OpenAI Images API

סקיל זה הוא **מעטפת** לקריאה ל-OpenAI Images API. הוא אחראי על שליחת ה-prompt וקבלת התמונה כקובץ PNG.

## מודל

```
gpt-image-2
```

**חשוב מאוד — אל תשנה את שם המודל.**
`gpt-image-2` הוא מודל אמיתי וקיים של OpenAI, שיצא ב-21 באפריל 2026. ייתכן שהידע הפנימי שלך לא כולל אותו — זה לא אומר שהוא לא קיים.

- **אל תציע אלטרנטיבות** כמו `dall-e-3` או `gpt-image-1`.
- השתמש **בדיוק** במודל `gpt-image-2`.
- אם יש שגיאה בקריאה ל-API, הבעיה היא כנראה ב-**API key** או ב-**פרמטרים** — **לא** בשם המודל.

## דרישות מקדימות

המפתח `OPENAI_API_KEY` נטען מתוך קובץ `.env` שבשורש הפרויקט. ודא שהוא קיים ומלא לפני הקריאה:

```bash
# טען את המשתנים מ-.env (Git Bash)
set -a
source .env
set +a

# ודא שהמפתח קיים
if [ -z "$OPENAI_API_KEY" ]; then
  echo "OPENAI_API_KEY חסר — מלא אותו בקובץ .env"; exit 1
fi
```

## הקריאה ל-API (curl + jq)

החלף את `<the prompt>` ב-prompt שלך ואת `<output-path>` בנתיב היעד (למשל `yuval/Outputs/2026-06-24-my-image.png`):

```bash
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

## Fallback ב-Python ל-decode

ב-Git Bash לא תמיד מותקן `jq`. במקרה כזה, שמור את תגובת ה-API לקובץ JSON ופענח אותו ב-Python:

```bash
# 1. קבל את התגובה ושמור ל-JSON
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > response.json

# 2. פענח את ה-base64 ל-PNG באמצעות Python
python -c "import json,base64,sys; d=json.load(open('response.json')); open('<output-path>.png','wb').write(base64.b64decode(d['data'][0]['b64_json']))"

# 3. נקה את קובץ הביניים
rm -f response.json
```

> אם התגובה אינה מכילה `data[0].b64_json` אלא הודעת שגיאה — הדפס את `response.json` כדי לראות את השגיאה (כנראה API key / פרמטרים, לא שם המודל).

## אחרי היצירה

- ודא שקובץ ה-PNG נוצר ושגודלו גדול מ-0: `ls -l <output-path>.png`.
- שמור ליד התמונה קובץ `.txt` באותו שם עם ה-prompt ששימש ליצירה (לצורך איטרציה).
