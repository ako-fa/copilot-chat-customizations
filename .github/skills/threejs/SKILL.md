---
name: threejs
description: "Three.js 3D グラフィックスの再利用可能なコードパターン集"
---

# Three.js Skill

Three.js 0.182.0 を使用した 3D シーン構築の再利用可能なコードパターン。

## 基本原則

- シーン管理は `useThreeScene` などの composable に集約し、初期化と破棄の責務を分離する
- ジオメトリとマテリアルは再利用し、不必要なインスタンス生成を避ける
- アニメーションループは `requestAnimationFrame` で管理し、`onUnmounted` で必ず停止する
- `onUnmounted` で `geometry.dispose()`、`material.dispose()`、`texture.dispose()`、`renderer.dispose()` を確実に実行する
- キャンバス要素は ref 経由で扱い、マウント時に追加、アンマウント時に DOM から除去する
- イベントリスナーは登録と解除を対で実装し、ライフサイクル外に残さない

## シーン初期化 Composable パターン

```typescript
// composables/useThreeScene.ts
import * as THREE from "three";

/**
 * Three.js シーンのライフサイクル管理を提供する Composable
 *
 * @param containerRef - シーンをマウントする HTML 要素の ref
 * @returns scene, camera, renderer の readonly 参照
 */
export const useThreeScene = (containerRef: Ref<HTMLElement | null>) => {
  let scene: THREE.Scene | null = null;
  let camera: THREE.PerspectiveCamera | null = null;
  let renderer: THREE.WebGLRenderer | null = null;
  let animationId: number | null = null;
  let removeResizeListener: (() => void) | null = null;

  const init = (): void => {
    if (!containerRef.value) return;

    const width = containerRef.value.clientWidth;
    const height = containerRef.value.clientHeight;

    // シーン設定
    scene = new THREE.Scene();
    scene.background = new THREE.Color(0x000000);

    // カメラ設定
    camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
    camera.position.z = 5;

    // レンダラー設定
    renderer = new THREE.WebGLRenderer({
      antialias: true,
      alpha: true,
    });
    renderer.setSize(width, height);
    renderer.setPixelRatio(window.devicePixelRatio);
    containerRef.value.appendChild(renderer.domElement);

    // リサイズ対応
    const onResize = (): void => {
      if (!containerRef.value || !camera || !renderer) return;
      const newWidth = containerRef.value.clientWidth;
      const newHeight = containerRef.value.clientHeight;
      camera.aspect = newWidth / newHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(newWidth, newHeight);
    };
    window.addEventListener("resize", onResize);
    removeResizeListener = () => window.removeEventListener("resize", onResize);
  };

  const destroy = (): void => {
    // アニメーションループ停止
    if (animationId !== null) {
      cancelAnimationFrame(animationId);
      animationId = null;
    }

    // リサイズリスナー解除
    removeResizeListener?.();

    // レンダラー破棄
    if (renderer && containerRef.value) {
      containerRef.value.removeChild(renderer.domElement);
      renderer.dispose();
    }

    scene = null;
    camera = null;
    renderer = null;
  };

  onMounted(init);
  onUnmounted(destroy);

  return {
    scene: readonly(computed(() => scene)),
    camera: readonly(computed(() => camera)),
    renderer: readonly(computed(() => renderer)),
  };
};
```

## ジオメトリ再利用パターン

```typescript
// 同じ形状を複数のメッシュで共有する
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material1 = new THREE.MeshPhongMaterial({ color: 0xff0000 });
const material2 = new THREE.MeshPhongMaterial({ color: 0x00ff00 });

// geometry は 1 インスタンスを再利用、material は異なる
const mesh1 = new THREE.Mesh(geometry, material1);
const mesh2 = new THREE.Mesh(geometry, material2);

// 不要になったら確実に .dispose() する
onUnmounted(() => {
  geometry.dispose(); // GPU メモリ解放
  material1.dispose();
  material2.dispose();
});
```

## アニメーションループパターン

```typescript
// requestAnimationFrame でレンダリングループを管理
let animationId: number | null = null;

const animate = (): void => {
  animationId = requestAnimationFrame(animate);

  // アニメーション更新処理
  mesh.rotation.x += 0.01;
  mesh.rotation.y += 0.01;

  renderer.render(scene, camera);
};

// 開始
onMounted(() => {
  animate();
});

// 停止（メモリリーク防止のため必須）
onUnmounted(() => {
  if (animationId !== null) {
    cancelAnimationFrame(animationId);
  }
});
```

## リソース破棄チェックリスト

`onUnmounted` で以下を確実に実行する：

```typescript
onUnmounted(() => {
  // 1. アニメーションループ停止
  if (animationId !== null) cancelAnimationFrame(animationId);

  // 2. ジオメトリ破棄
  geometry.dispose();

  // 3. マテリアル破棄
  material.dispose();

  // 4. テクスチャ破棄（使用している場合）
  texture?.dispose();

  // 5. レンダラー破棄
  renderer.dispose();

  // 6. DOM からキャンバスを除去
  containerRef.value?.removeChild(renderer.domElement);

  // 7. イベントリスナー解除
  window.removeEventListener("resize", onResize);
});
```

## マウスインタラクションパターン

```typescript
// composables/useMouseTracking.ts

/**
 * マウス位置のトラッキングを提供する Composable
 *
 * @returns mouseX, mouseY の readonly ref
 */
export const useMouseTracking = (): {
  mouseX: Readonly<Ref<number>>;
  mouseY: Readonly<Ref<number>>;
} => {
  const mouseX = ref(0);
  const mouseY = ref(0);

  const onMouseMove = (event: MouseEvent): void => {
    mouseX.value = event.clientX;
    mouseY.value = event.clientY;
  };

  onMounted(() => window.addEventListener("mousemove", onMouseMove));
  onUnmounted(() => window.removeEventListener("mousemove", onMouseMove));

  return {
    mouseX: readonly(mouseX),
    mouseY: readonly(mouseY),
  };
};
```

## 実施手順

1. `useThreeScene` composable でシーン・カメラ・レンダラーを初期化する
2. ジオメトリとマテリアルを定義し、再利用可能な構成にする
3. `requestAnimationFrame` でアニメーションループを設定する
4. マウスインタラクションなどのイベントを接続する
5. `onUnmounted` ですべてのリソース破棄とイベント解除を実装する

## チェックリスト

- シーン初期化・リサイズ対応・破棄が composable 内で一元管理されているか
- ジオメトリ再利用が実装され、不要なインスタンス生成をしていないか
- アニメーションループが `requestAnimationFrame` で動作し、アンマウント時に停止しているか
- `geometry`、`material`、`texture`、`renderer` の `dispose()` を `onUnmounted` で実行しているか
- キャンバス要素を DOM へ追加した後、アンマウント時に確実に除去しているか
- `resize` や `mousemove` などのイベントリスナーを適切に解除しているか
