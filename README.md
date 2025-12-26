# AI-Whispering-ASMR-Generation
A comprehensive guide to creating natural, non-robotic Whispering ASMR using Amazon Polly's Standard TTS. Learn how to leverage SSML tags to simulate human-like breathing and pauses, overcoming the stiffness of typical AI speech generation.

## 🚀 Overview
The generation process consists of two main phases:

- **Voice Synthesis:** Utilizing Amazon Polly's Standard TTS with SSML for "whispering" effects.
- **Spatial Post-Processing:** Using Unity to simulate binaural 3D audio dynamics to mimic realistic ear-to-ear movement.

## 🛠️ Technology Stack & Rationale
**1. Text-to-Speech (TTS) Engine**
- **Tool:** Amazon Polly (Standard Engine)
- **Technique:** Speech Synthesis Markup Language (SSML)

**Why Standard TTS over Neural TTS?**
While Neural TTS often offers smoother flow, we selected the Standard TTS engine for this pipeline after extensive comparison.
- **SSML Support:** It allows for granular control via SSML tags (specifically <amazon:effect name="whispered">), which is essential for achieving a pure whispering timbre.
- **Prosody Simulation:** We can explicitly program human-like pauses and breathing patterns (<amazon:breath>) to enhance realism.
- **Consistency:** Standard TTS minimizes the variance in acoustic characteristics across different languages, eliminating "AI hallucinations" or inconsistencies often found in neural models during cross-language generation.

**2. Spatial Audio Simulation**
- **Tool:** Unity 3D Engine
- **Objective:** To simulate the spatial immersion typical of ASMR triggers (Binaural Audio).

**Addressing the Anticipation Effect:** To prevent the listener from predicting the audio movement (which can diminish the ASMR response), we implemented a randomized movement algorithm rather than a fixed loop.

## ⚙️ Implementation Details
**Step 1: SSML Configuration**
We utilize SSML tags to modulate the voice output. The script must be wrapped in the whispering effect tag.

Example Structure:
```
XML
<speak>
<s><prosody rate="x-slow"><amazon:effect name="whispered" >Welcome, to Ninty-nine ASMR</amazon:effect></prosody></s>
<amazon:breath duration="long" volume="x-soft"/>
<break time="1s"/>
</speak>
```
            
**Step 2: Spatial Randomization (Unity)**
The audio is processed in a 3D environment to pan around the listener's head. The movement logic is randomized to ensure unpredictability.

**Parameter Settings:**

- Panning Duration (Speed): Randomized between 1.0s and 3.0s for travel between ears.
```
private void ResetMovement()
    {
        currentTime = 0f;
        totalTimeForHalfCircle = Random.Range(1f, 3f);
        float temp = startAngle;
        startAngle = endAngle;
        endAngle = temp == 0 ? 180f : 0f;
        isMoving = true;
    }
```
- Hold Duration (Dwell time): Randomized between 0s and 3.0s at the endpoints before changing direction.
```
void Update()
    {
        if (isPausing)
        {
            pauseTimer += Time.deltaTime;
            if (pauseTimer >= pauseTime)
            {
                isPausing = false;
                pauseTimer = 0f;
                ResetMovement();
            }
        }
        else if (isMoving)
        {
            currentTime += Time.deltaTime;
            float fractionOfJourney = currentTime / totalTimeForHalfCircle;
            float currentAngle = Mathf.Lerp(startAngle, endAngle, fractionOfJourney) * Mathf.Deg2Rad;
            float x = Mathf.Cos(currentAngle) * radius + cameraTransform.position.x;
            float z = Mathf.Sin(currentAngle) * radius + cameraTransform.position.z;
            transform.position = new Vector3(x, transform.position.y, z);
            transform.LookAt(new Vector3(cameraTransform.position.x, transform.position.y, cameraTransform.position.z));

            if (currentTime >= totalTimeForHalfCircle)
            {
                isMoving = false;
                pauseTime = Random.Range(0f, 3f);
                isPausing = true;
            }
        }
    }
```
