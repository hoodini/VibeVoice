<div align="center">

# 🎙️ VibeVoice - מערכת המרת טקסט לדיבור בזמן אמת

[![Hugging Face](https://img.shields.io/badge/HuggingFace-Collection-orange?logo=huggingface)](https://huggingface.co/collections/microsoft/vibevoice-68a2ef24a875c44be47b034f)
[![Colab](https://img.shields.io/badge/נסה%20ב-Colab-orange?logo=googlecolab)](https://colab.research.google.com/github/microsoft/VibeVoice/blob/main/demo/vibevoice_realtime_colab.ipynb)

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="Figures/VibeVoice_logo_white.png">
  <img src="Figures/VibeVoice_logo.png" alt="VibeVoice Logo" width="300">
</picture>

**מערכת AI מתקדמת של מיקרוסופט להפיכת טקסט לדיבור אנושי טבעי**

</div>

---

## 📖 מה זה VibeVoice? (בפשטות)

דמיינו שיש לכם "קורא" אוטומטי שיודע לקרוא כל טקסט בקול אנושי טבעי - זה בדיוק מה ש-VibeVoice עושה!

### ⚡ יכולות עיקריות

| יכולת | הסבר |
|--------|------|
| **זמן אמת** | מתחיל לדבר תוך ~300 מילישניות |
| **סטרימינג** | לא צריך לחכות שכל הטקסט יעובד |
| **קולות מגוונים** | גברים, נשים, שפות שונות |
| **איכות גבוהה** | קול טבעי שנשמע כמו אדם אמיתי |

---

## 🎯 דוגמאות שימוש מעשיות

### 1. 🤖 צ'אט בוט מדבר
```
משתמש: "מה מזג האוויר היום?"
→ הבוט לא רק מציג טקסט, אלא גם מדבר את התשובה בקול!
```

### 2. 📚 הקראת ספרים
```
קובץ טקסט של ספר → קובץ אודיו מוכן להאזנה
(יצירת אודיובוקים אוטומטית)
```

### 3. ♿ נגישות
```
אתר אינטרנט → הקראה קולית לאנשים עם לקויות ראייה
```

### 4. 🎙️ פודקאסטים
```
תסריט כתוב → פודקאסט מוכן עם קולות שונים
```

### 5. 📞 מוקדי שירות
```
מענה קולי אוטומטי → תגובות טבעיות ללקוחות
```

---

## 🏗️ איך המערכת עובדת? (תרשים מפורט)

```mermaid
flowchart TB
    subgraph INPUT ["📥 קלט"]
        TEXT["📝 טקסט לקריאה<br/>(למשל: 'שלום עולם')"]
        VOICE["🎤 בחירת קול<br/>(Carter, Emma, וכו')"]
    end

    subgraph PROCESSOR ["⚙️ מעבד (Processor)"]
        direction TB
        TOKENIZER["🔤 Tokenizer<br/>המרת טקסט למספרים"]
        CACHED_PROMPT["💾 Voice Cache<br/>מאפייני הקול הנבחר"]
        
        TOKENIZER --> BATCH_ENCODE["📦 Batch Encoding<br/>ארגון הנתונים"]
        CACHED_PROMPT --> BATCH_ENCODE
    end

    subgraph MODEL ["🧠 המודל (Model)"]
        direction TB
        
        subgraph LM ["Language Model - הבנת הטקסט"]
            TEXT_LM["📖 Language Model<br/>(מבוסס Qwen2.5)<br/>מבין את הטקסט"]
        end
        
        subgraph TTS_LM ["TTS Language Model - יצירת דיבור"]
            TTS_MODEL["🗣️ TTS Language Model<br/>שכבות עליונות ליצירת דיבור"]
            EOS_CLASSIFIER["🔚 EOS Classifier<br/>זיהוי סוף משפט"]
        end
        
        subgraph DIFFUSION ["Diffusion Head - יצירת הצליל"]
            NOISE["🌫️ רעש אקראי"]
            TIMESTEP["⏱️ Timestep Embedder<br/>קידוד זמן"]
            CONDITION["🎯 Condition<br/>תנאי הדיבור"]
            HEAD_LAYERS["🔄 Head Layers<br/>שכבות עיבוד"]
            FINAL["🎵 Final Layer<br/>שכבה סופית"]
            
            NOISE --> HEAD_LAYERS
            TIMESTEP --> HEAD_LAYERS
            CONDITION --> HEAD_LAYERS
            HEAD_LAYERS --> FINAL
        end
        
        TEXT_LM --> TTS_MODEL
        TTS_MODEL --> DIFFUSION
        TTS_MODEL --> EOS_CLASSIFIER
    end

    subgraph DECODER ["🔊 מפענח האודיו"]
        direction TB
        ACOUSTIC["🎹 Acoustic Tokenizer<br/>המרה לצליל"]
        STREAMING_CACHE["📚 Streaming Cache<br/>זיכרון לסטרימינג"]
        AUDIO_DECODER["🔈 Audio Decoder<br/>יצירת גל קולי"]
        
        ACOUSTIC --> STREAMING_CACHE
        STREAMING_CACHE --> AUDIO_DECODER
    end

    subgraph OUTPUT ["📤 פלט"]
        STREAMER["🌊 Audio Streamer<br/>שידור חי של האודיו"]
        WAV_FILE["📁 קובץ WAV<br/>שמירה לקובץ"]
        WEBSOCKET["🌐 WebSocket<br/>שידור לדפדפן"]
    end

    TEXT --> PROCESSOR
    VOICE --> PROCESSOR
    PROCESSOR --> MODEL
    MODEL --> DECODER
    DECODER --> OUTPUT

    style INPUT fill:#e1f5fe
    style PROCESSOR fill:#fff3e0
    style MODEL fill:#f3e5f5
    style DECODER fill:#e8f5e9
    style OUTPUT fill:#fce4ec
```

---

## 📁 מבנה הפרויקט

```mermaid
flowchart LR
    subgraph ROOT ["📂 VibeVoice"]
        direction TB
        
        subgraph DEMO ["📂 demo - דוגמאות הרצה"]
            REALTIME_DEMO["vibevoice_realtime_demo.py<br/>🌐 הפעלת שרת WebSocket"]
            FILE_DEMO["realtime_model_inference_from_file.py<br/>📁 המרת קובץ טקסט"]
            COLAB["vibevoice_realtime_colab.ipynb<br/>☁️ נוטבוק לגוגל קולאב"]
            
            subgraph VOICES ["📂 voices - קבצי קולות"]
                VOICE_FILES["🎤 קבצי .pt<br/>Carter, Emma, Frank..."]
            end
            
            subgraph WEB ["📂 web - ממשק אינטרנט"]
                APP["app.py<br/>🖥️ שרת FastAPI"]
                HTML["index.html<br/>🌐 דף הבית"]
            end
        end
        
        subgraph VIBEVOICE ["📂 vibevoice - קוד המודל"]
            direction TB
            
            subgraph MODULAR ["📂 modular - רכיבי המודל"]
                CONFIG["configuration_*.py<br/>⚙️ הגדרות"]
                STREAMING["modeling_vibevoice_streaming*.py<br/>🧠 המודל הראשי"]
                DIFFUSION["modular_vibevoice_diffusion_head.py<br/>🎵 ראש הדיפוזיה"]
                TOKENIZER_MOD["modular_vibevoice_tokenizer.py<br/>🔤 טוקנייזר"]
                STREAMER_MOD["streamer.py<br/>🌊 סטרימר"]
            end
            
            subgraph PROCESSOR_DIR ["📂 processor - עיבוד קלט/פלט"]
                STREAMING_PROC["vibevoice_streaming_processor.py<br/>⚙️ מעבד סטרימינג"]
                TOKENIZER_PROC["vibevoice_tokenizer_processor.py<br/>🔤 מעבד אודיו"]
            end
            
            subgraph SCHEDULE ["📂 schedule - תזמון"]
                DPM["dpm_solver.py<br/>📐 DPM Solver"]
            end
        end
    end

    style DEMO fill:#e3f2fd
    style VIBEVOICE fill:#f3e5f5
    style MODULAR fill:#fff3e0
    style PROCESSOR_DIR fill:#e8f5e9
```

---

## 🔄 זרימת הנתונים המלאה

```mermaid
sequenceDiagram
    participant U as 👤 משתמש
    participant P as ⚙️ Processor
    participant LM as 📖 Language Model
    participant TTS as 🗣️ TTS Model
    participant D as 🎵 Diffusion Head
    participant A as 🔊 Acoustic Decoder
    participant S as 🌊 Streamer

    U->>P: טקסט + בחירת קול
    P->>P: טוקניזציה (המרה למספרים)
    P->>LM: טוקנים מוכנים
    
    loop עבור כל חלון טקסט (5 טוקנים)
        LM->>LM: הבנת הקשר הטקסט
        LM->>TTS: Hidden States
        
        loop עבור כל טוקן דיבור (6 פעמים)
            TTS->>D: תנאים ליצירת צליל
            D->>D: תהליך דיפוזיה (5 צעדים)
            D->>A: Latent מוכן
            A->>S: קטע אודיו קצר
            S->>U: 🔈 אודיו בזמן אמת!
        end
        
        TTS->>TTS: בדיקת EOS (סוף?)
    end
    
    S->>U: ✅ סיום
```

---

## 🔗 קשרים בין הרכיבים

```mermaid
graph TB
    subgraph ENTRY ["🚪 נקודות כניסה"]
        CLI["💻 Command Line<br/>python demo/realtime_model_inference_from_file.py"]
        WEB_UI["🌐 Web Interface<br/>python demo/vibevoice_realtime_demo.py"]
        API["🔌 API<br/>WebSocket /stream"]
    end

    subgraph CORE ["🎯 ליבת המערכת"]
        direction TB
        
        PROC["VibeVoiceStreamingProcessor<br/>━━━━━━━━━━━━━━━━━━━<br/>• from_pretrained()<br/>• process_input_with_cached_prompt()<br/>• save_audio()"]
        
        MODEL_INF["VibeVoiceStreamingForConditionalGenerationInference<br/>━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━<br/>• forward_lm() - הבנת טקסט<br/>• forward_tts_lm() - יצירת דיבור<br/>• generate() - הרצה מלאה<br/>• sample_speech_tokens() - דיפוזיה"]
        
        STREAMING_MODEL["VibeVoiceStreamingModel<br/>━━━━━━━━━━━━━━━━━━<br/>• language_model (Qwen2.5)<br/>• tts_language_model<br/>• acoustic_tokenizer<br/>• prediction_head"]
    end

    subgraph COMPONENTS ["🧩 רכיבים"]
        DIFF_HEAD["VibeVoiceDiffusionHead<br/>━━━━━━━━━━━━━━━━<br/>• TimestepEmbedder<br/>• HeadLayers<br/>• FinalLayer"]
        
        TOKENIZER["VibeVoiceAcousticTokenizerModel<br/>━━━━━━━━━━━━━━━━━━━━━<br/>• TokenizerDecoder<br/>• SConv1d (עם cache)<br/>• SConvTranspose1d"]
        
        AUDIO_STREAM["AudioStreamer<br/>━━━━━━━━━━<br/>• put() - הכנסת אודיו<br/>• end() - סיום<br/>• get_stream() - קבלת זרם"]
        
        DPM_SOLVER["DPMSolverMultistepScheduler<br/>━━━━━━━━━━━━━━━━━━━━<br/>• set_timesteps()<br/>• step()"]
    end

    CLI --> PROC
    WEB_UI --> PROC
    API --> PROC
    
    PROC --> MODEL_INF
    MODEL_INF --> STREAMING_MODEL
    
    STREAMING_MODEL --> DIFF_HEAD
    STREAMING_MODEL --> TOKENIZER
    MODEL_INF --> AUDIO_STREAM
    MODEL_INF --> DPM_SOLVER

    style ENTRY fill:#e1f5fe
    style CORE fill:#f3e5f5
    style COMPONENTS fill:#e8f5e9
```

---

## 🚀 התקנה והרצה

### דרישות מקדימות
- Python 3.9+
- CUDA (מומלץ) או MPS (Mac) או CPU
- ~4GB זיכרון GPU

### התקנה

```bash
# שכפול הפרויקט
git clone https://github.com/microsoft/VibeVoice.git
cd VibeVoice

# התקנת החבילה
pip install -e .
```

### הרצת דמו בזמן אמת (WebSocket)

```bash
python demo/vibevoice_realtime_demo.py --model_path microsoft/VibeVoice-Realtime-0.5B
```

פתחו את הדפדפן בכתובת `http://localhost:3000`

### המרת קובץ טקסט

```bash
python demo/realtime_model_inference_from_file.py \
    --model_path microsoft/VibeVoice-Realtime-0.5B \
    --txt_path demo/text_examples/1p_vibevoice.txt \
    --speaker_name Carter
```

---

## 🎤 קולות זמינים

| שפה | קולות גברים | קולות נשים |
|-----|-------------|-------------|
| 🇺🇸 אנגלית | Carter, Davis, Frank, Mike | Emma, Grace |
| 🇩🇪 גרמנית | de-Spk0 | de-Spk1 |
| 🇫🇷 צרפתית | fr-Spk0 | fr-Spk1 |
| 🇮🇹 איטלקית | it-Spk1 | it-Spk0 |
| 🇯🇵 יפנית | jp-Spk0 | jp-Spk1 |
| 🇰🇷 קוריאנית | kr-Spk1 | kr-Spk0 |
| 🇳🇱 הולנדית | nl-Spk0 | nl-Spk1 |
| 🇵🇱 פולנית | pl-Spk0 | pl-Spk1 |
| 🇵🇹 פורטוגזית | pt-Spk1 | pt-Spk0 |
| 🇪🇸 ספרדית | sp-Spk1 | sp-Spk0 |
| 🇮🇳 הינדית | Samuel | - |

---

## 📊 ארכיטקטורת המודל

```mermaid
graph TB
    subgraph Architecture ["🏛️ ארכיטקטורת VibeVoice-Realtime-0.5B"]
        direction TB
        
        subgraph Params ["📊 פרמטרים"]
            SIZE["גודל: 0.5B פרמטרים"]
            CONTEXT["אורך הקשר: 8K טוקנים"]
            GENERATION["אורך יצירה: ~10 דקות"]
            LATENCY["השהייה ראשונה: ~300ms"]
        end
        
        subgraph Tech ["🔧 טכנולוגיות"]
            BASE["בסיס: Qwen2.5"]
            DIFFUSION_TECH["דיפוזיה: DPM-Solver++"]
            ACOUSTIC_TECH["אקוסטי: 7.5Hz frame rate"]
            SAMPLE["דגימה: 24kHz"]
        end
        
        subgraph Process ["⚙️ תהליך"]
            WINDOW["חלון טקסט: 5 טוקנים"]
            SPEECH["חלון דיבור: 6 טוקנים"]
            STEPS["צעדי דיפוזיה: 5"]
            CFG["CFG Scale: 1.5"]
        end
    end
```

---

## ⚠️ מגבלות והתראות

### מגבלות טכניות
- 🔤 **שפות**: אנגלית עובדת הכי טוב, שפות אחרות ניסיוניות
- 📏 **טקסט קצר**: פחות מ-3 מילים עלול להיות לא יציב
- 🔢 **קוד ונוסחאות**: לא תומך בקריאת קוד או מתמטיקה
- 🎵 **רעשי רקע**: לא יוצר מוזיקה או אפקטים קוליים

### אזהרות חשובות
⚠️ **סיכוני Deepfake**: המערכת יכולה ליצור קולות משכנעים. השתמשו באחריות!

⚠️ **שימוש מחקרי בלבד**: לא מומלץ לשימוש מסחרי ללא בדיקות נוספות.

⚠️ **גילוי נאות**: תמיד ציינו כשמשתמשים בתוכן שנוצר ע"י AI.

---

## 📄 רישיון

ראו קובץ [LICENSE](LICENSE) לפרטים.

---

## 🔗 קישורים

- [דף הפרויקט](https://microsoft.github.io/VibeVoice)
- [HuggingFace](https://huggingface.co/microsoft/VibeVoice-Realtime-0.5B)
- [דוח טכני](https://arxiv.org/pdf/2508.19205)
- [נסו ב-Colab](https://colab.research.google.com/github/microsoft/VibeVoice/blob/main/demo/vibevoice_realtime_colab.ipynb)

---

<div align="center">

**נוצר על ידי צוות VibeVoice במיקרוסופט** 🎙️

</div>
