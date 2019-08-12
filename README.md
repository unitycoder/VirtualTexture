# VirtualTexture

## Introduction
Virtual texture for unity.

## Run The Demo
1. Open the project with Unity 2018(or later).
2. Click "Menu -> Demo -> Generate Virtual Texture" to generate needed data, it may takes about 10 minutes.
3. Open Demo Scene.
4. Play. Use w/s/a/d and mouse to move the camera.

*Screen Shot*
| far away | ![Far Screenhot](/Image/Screenhot1.png)   |
| -------- | ------------------------------------------|
| close up | ![Close Screenhot](/Image/Screenhot2.png) |

## Implement Detail

### Feedback Pass
##### 1. Feedback Renderer

Render the scene with feedback shader:

```c++
fixed4 VTFragFeedback(VTV2f i) : SV_Target
{
	float2 page = floor(i.uv * _VTFeedbackParam.x);
	float2 uv = i.uv * _VTFeedbackParam.y;
	float2 dx = ddx(uv);
	float2 dy = ddy(uv);
	int mip = clamp(int(0.5 * log2(max(dot(dx, dx), dot(dy, dy))) + 0.5 + _VTFeedbackParam.w), 0, _VTFeedbackParam.z);

	return fixed4(page / 255.0, mip / 255.0, 1);
}
```

![Feedback Renderer](/Image/FeedbackRenderer.png)

##### 2. Feedback Reader

Readback feedback texture from gpu to cpu.

![Feedback Reader](/Image/FeedbackReader.png)

##### 3. Tiled Texture

Load all need texture tiles into TiledTexture.

![Tiled Texture](/Image/TiledTexture.png)

##### 4. Page Table

update page table and lookup texture.

![Page Table](/Image/PageTable.png)

### Shading Pass

```c++
fixed4 VTFragUnlit(VTV2f i) : SV_Target
{
	float2 uv = VTTransferUV(i.uv);
	fixed4 col = VTTex2D0(uv);
	return col;
}

float2 VTTransferUV(float2 uv)
{
	float2 uvInt = uv - frac(uv * _VTPageParam.x) * _VTPageParam.y;
	fixed4 page = tex2D(_VTLookupTex, uvInt) * 255;
	float2 inPageOffset = frac(uv * exp2(_VTPageParam.z - page.b));
	float2 inTileOffset = inPageOffset * _VTTileParam.y + _VTTileParam.x;
	return (page.rg + inTileOffset) * _VTTileParam.zw;
}

fixed4 VTTex2D0(float2 uv)
{
	return tex2D(_VTTiledTex0, uv);
}
```

## Reference
http://www.silverspaceship.com/src/svt/
