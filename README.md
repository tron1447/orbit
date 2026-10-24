[index.tsx](https://github.com/user-attachments/files/33901628/index.tsx)import { createFileRoute } from "@tanstack/react-router";
import { useEffect, useState } from "react";
import Game, { xrStore } from "@/game/Game";

export const Route = createFileRoute("/")({
  ssr: false,
  head: () => ({
    meta: [
      { title: "Starfall — VR space exploration" },
      { name: "description", content: "Fly between alien planets, land, scan strange life and mine crystals — in your VR headset browser." },
      { property: "og:title", content: "Starfall — VR space exploration" },
      { property: "og:description", content: "A No Man's Sky-style VR game that runs in your headset's browser." },
      { property: "og:type", content: "website" },
      { name: "twitter:card", content: "summary_large_image" },
    ],
  }),
  component: Index,
});

function Index() {
  const [supported, setSupported] = useState<boolean | null>(null);
  useEffect(() => {
    navigator.xr?.isSessionSupported("immersive-vr").then(setSupported).catch(() => setSupported(false));
    if (!navigator.xr) setSupported(false);
  }, []);
  return (
    <div className="fixed inset-0 bg-background">
      <Game />
      <div className="pointer-events-none absolute inset-0 flex flex-col items-center justify-center gap-6 bg-gradient-to-b from-transparent to-background/80 p-6 text-center">
        <h1 className="font-display text-6xl tracking-[0.3em] text-primary md:text-8xl">STARFALL</h1>
        <p className="max-w-md text-foreground/80">Fly between planets, land, scan alien life and mine crystals.</p>
        <button
          disabled={!supported}
          onClick={() => xrStore.enterVR()}
          className="pointer-events-auto rounded-full bg-primary px-10 py-4 text-lg font-semibold text-primary-foreground shadow-lg transition hover:scale-105 disabled:opacity-40"
        >
          Enter VR
        </button>
        {supported === false && <p className="text-sm text-muted-foreground">Open this page in your VR headset's browser to play.</p>}
        <ul className="max-w-lg text-left text-sm text-foreground/70">
          <li>Space: left stick pitch/roll · right stick yaw · right trigger thrust · grip brake · A land near a planet</li>
          <li>Planet: left stick walk · right stick snap turn · left trigger scan plants · hold right trigger on crystals to mine · B by your ship to launch</li>
          <li>Y: turn 10 carbon into fuel · wrist menu on your left hand</li>
        </ul>
      </div>
    </div>
  );
}

