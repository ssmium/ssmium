import React, { useEffect, useState } from "react";
import { Globe as Globe2, SunMedium, MoonStar } from "lucide-react";

// ===== THEME =====
const THEME = {
  deep: "#001A33",
  deep2: "#00264D",
  light: "#3399FF",
  text: "#111111",
  textSub: "#333333",
  bg: "#FFFFFF",
  bgSoft: "#F5F7FA",
};

// ===== NAV =====
const NAV = [
  { id: "home", label: "HOME" },
  { id: "about", label: "회사소개" },
  { id: "services", label: "사업분야" },
  { id: "news", label: "소식" },
  { id: "contact", label: "안내" },
];

// ===== ROUTES =====
const ROUTES = {
  HOME: "home",
  ABOUT: "about",
  SERVICES: "services",
  NEWS: "news",
  CONTACT: "contact",
  NEWS_DETAIL: "news_detail",
  SERVICE_DETAIL: "service_detail",
};

// ===== ROOT APP =====
export default function HypokeimenonSite() {
  const [dark, setDark] = useState(false);
  const [route, setRoute] = useState(ROUTES.HOME);
  const [selectedNews, setSelectedNews] = useState(null);
  const [selectedService, setSelectedService] = useState(null);

  // Smoke tests
  useEffect(() => {
    console.assert(document.querySelector("main") !== null, "[TEST] main exists");
    console.assert(Array.isArray(NAV) && NAV.length >= 5, "[TEST] NAV present");
  }, []);

  const renderRoute = () => (
    <>
      {route === ROUTES.HOME && <HomePage onCTA={() => setRoute(ROUTES.ABOUT)} />}
      {route === ROUTES.ABOUT && <AboutPage goServices={() => setRoute(ROUTES.SERVICES)} />}
      {route === ROUTES.SERVICES && <ServicesPage />}
      {route === ROUTES.SERVICE_DETAIL && (
        <ServiceDetailPage service={selectedService} onBack={() => setRoute(ROUTES.SERVICES)} />
      )}
      {route === ROUTES.NEWS && <NewsPage onOpen={(post) => setSelectedNews(post)} />}
      {route === ROUTES.NEWS_DETAIL && (
        <NewsDetailPage post={selectedNews} onBack={() => setRoute(ROUTES.NEWS)} />
      )}
      {route === ROUTES.CONTACT && <ContactPage />}
    </>
  );

  return (
    <div style={{ background: THEME.bg, color: THEME.text }}>
      <Header dark={dark} setDark={setDark} setRoute={setRoute} />
      <main>{renderRoute()}</main>
      <Footer setRoute={setRoute} />
    </div>
  );
}

// ===== HEADER =====
function Header({ dark, setDark, setRoute }) {
  return (
    <header
      className="sticky top-0 z-50 backdrop-blur border-b"
      style={{ background: "rgba(255,255,255,0.9)", borderColor: "#E6EDF5" }}
    >
      <div className="mx-auto max-w-7xl px-4 py-3 flex items-center justify-between">
        {/* Brand */}
        <button onClick={() => setRoute(ROUTES.HOME)} className="flex items-center gap-2" id="brand">
          <div className="h-8 w-8 rounded grid place-items-center text-white" style={{ background: THEME.deep2 }}>
            <Globe2 className="h-4 w-4" />
          </div>
          <div className="text-left">
            <span className="block text-sm font-bold" style={{ color: THEME.deep2 }}>
              HYPOKEIMENON
            </span>
            <span className="block text-xs text-slate-500">히포케이메논</span>
          </div>
        </button>

        {/* Nav */}
        <nav className="hidden md:flex items-center gap-6 text-sm">
          {NAV.map((n) => (
            <button
              key={n.id}
              onClick={() => setRoute(n.id)}
              className="hover:opacity-80"
              style={{ color: THEME.textSub }}
            >
              {n.label}
            </button>
          ))}
        </nav>

        {/* Theme toggle */}
        <button
          onClick={() => setDark(!dark)}
          className="rounded-full px-3 py-2 text-xs border"
          style={{ borderColor: "#E6EDF5" }}
          aria-label="Toggle theme"
        >
          {dark ? <SunMedium className="h-4 w-4" /> : <MoonStar className="h-4 w-4" />}
        </button>
      </div>
    </header>
  );
}

// ===== HOME =====
function HomePage({ onCTA }) {
  // HOME 커버: 구글 드라이브 이미지 사용 + 다중 후보 시도
  const coverRaw =
    "https://drive.google.com/file/d/1lM97utwPH3DzEccPRE3ywmj1MeVwi_lN/view?usp=sharing";
  const FALLBACK_URL =
    "https://images.unsplash.com/photo-1503264116251-35a269479413?q=80&w=1600&auto=format&fit=crop";

  const buildDriveCandidates = (raw, width = 1920) => {
    try {
      const url = new URL(raw);
      let id = null;
      const marker = "/file/d/";
      const idx = url.pathname.indexOf(marker);
      if (idx !== -1) {
        const rest = url.pathname.slice(idx + marker.length);
        id = rest.split("/")[0];
      }
      if (!id) id = url.searchParams.get("id");
      const resourcekey = url.searchParams.get("resourcekey");
      if (!id) return [raw];
      const rk = resourcekey ? `&resourcekey=${resourcekey}` : "";
      const baseView = `https://drive.google.com/uc?export=view&id=${id}${rk}`;
      const baseDownload = `https://drive.google.com/uc?export=download&id=${id}${rk}`;
      const basePlain = `https://drive.google.com/uc?id=${id}${rk}`;
      const thumb = `https://drive.google.com/thumbnail?id=${id}&sz=w${width}`;
      const proxy = `https://images.weserv.nl/?url=${encodeURIComponent(baseView)}`;
      return [baseDownload, baseView, basePlain, thumb, proxy];
    } catch {
      return [raw];
    }
  };

  const candidates = React.useMemo(() => buildDriveCandidates(coverRaw), [coverRaw]);
  const [heroAttempt, setHeroAttempt] = React.useState(0);
  const [heroSrc, setHeroSrc] = React.useState(candidates[0] || coverRaw);

  useEffect(() => {
    const h1 = document.querySelector('[data-test="hero-headline"]');
    console.assert(h1 && getComputedStyle(h1).fontSize, "[TEST] hero headline exists");
    const img = document.querySelector('[data-test="hero-img"]');
    console.assert(!!img, "[TEST] hero cover image exists");
  }, []);

  return (
    <section className="relative" aria-label="메인 배너">
      <div className="relative h-[76vh] overflow-hidden" style={{ background: THEME.deep }}>
        {/* 커버 이미지 */}
        <img
          data-test="hero-img"
          src={heroSrc}
          alt=""
          className="absolute inset-0 w-full h-full object-cover"
          referrerPolicy="no-referrer"
          crossOrigin="anonymous"
          onError={() => {
            if (heroAttempt < candidates.length - 1) {
              setHeroAttempt((n) => n + 1);
              setHeroSrc(candidates[heroAttempt + 1]);
            } else if (heroSrc !== FALLBACK_URL) {
              setHeroSrc(FALLBACK_URL);
            }
          }}
        />
        {/* 가독성을 위한 오버레이 */}
        <div
          className="absolute inset-0"
          style={{
            background:
              "linear-gradient(180deg, rgba(0,0,0,0.45), rgba(0,0,0,0.35) 40%, rgba(0,0,0,0.25))",
          }}
        />

        {/* 내용 */}
        <div className="relative z-10 h-full max-w-7xl mx-auto px-4 flex flex-col items-start justify-center text-white">
          <h1
            data-test="hero-headline"
            className="mt-3 font-extrabold leading-[1.15] tracking-tight"
            style={{ fontSize: "15pt" }}
          >
          </h1>
          <p className="mt-5 max-w-2xl text-lg text-slate-200">
            
          </p>
          <div className="mt-8">
            <button
              onClick={onCTA}
              className="rounded-full px-6 py-3 text-sm font-bold shadow border"
              style={{ background: THEME.light, borderColor: THEME.light, color: "#fff" }}
            >
              회사소개 보기
            </button>
          </div>
        </div>
      </div>
    </section>
  );
}

// ===== ABOUT (Upbit-like) =====
function AboutPage({ goServices }) {
  useEffect(() => {
    const items = document.querySelectorAll('[data-test="about-timeline"] li');
    console.assert(items.length >= 5, "[TEST] about timeline items rendered");
    const galleryImgs = document.querySelectorAll('[data-test="office-gallery"] img');
    console.assert(galleryImgs.length >= 3, "[TEST] office gallery rendered");
  }, []);

  const history = [
    { year: "2018", text: "07월, 비트코인 트레이딩 Team 설립" },
    { year: "2021", text: "02월 (주)히포케이메논 설립 / 03월 VVIP멤버십 / 06월 웰시클래스 / 09월 웰시투자그룹 / 12월 매출 98억" },
    { year: "2022", text: "08월 본사 확장 이전 / 09월 대한민국 우수기업 브랜드대상 1위" },
    { year: "2023", text: "02월 ICBA 브랜드혁신기업대상 1위 / 2023 한국소비자만족지수 1위" },
    { year: "2024", text: "09월 IBK 기업은행 동반자상 수상" },
    { year: "2025", text: "01월 테더백·02월 웰시투자그룹 한국소비자만족지수 1위 / 03월 웰시투자그룹 합정역 확장 이전" },
  ];

  return (
    <section className="mx-auto max-w-7xl px-4 py-20" aria-label="회사소개">
      <div className="grid lg:grid-cols-2 gap-10 items-center">
        {/* Left copy */}
        <div>
          <h2 className="text-3xl font-extrabold" style={{ color: THEME.text }}>히포케이메논</h2>
          <p className="mt-5 text-sm leading-7" style={{ color: THEME.textSub }}>
            히포케이메논은 2021년 2월에 설립된 커머스 마케팅 전문 기업입니다. 모든 브랜드 속에 내재된 불변의 진정성을 찾아내어, 끊임없이 변화하는 미디어 환경에서도 그 본질이 빛나도록 창의적인 방식으로 표현하는 데 있습니다.
          </p>
          <p className="mt-3 text-sm leading-7" style={{ color: THEME.textSub }}>
            설립 1년 차에 매출 98억이라는 놀라운 성장을 기록하며, 2년 차에는 매출 180억을 달성하는 쾌거를 이루었습니다.
          </p>
          <div className="mt-6 flex flex-wrap gap-2">
            {[
              { label: "웰시투자그룹", id: "wealthy", url: "http://wealthyinvestment.co.kr/" },
              { label: "테더백", id: "tetherback", url: "https://tetherback.io/" },
              { label: "가투법", id: "gatu", url: "https://cafe.naver.com/1djr58" },
            ].map((b) => (
              <a
                key={b.id}
                href={b.url}
                target="_blank"
                rel="noreferrer noopener"
                className="rounded px-4 py-2 text-xs font-bold border bg-white"
                style={{ borderColor: "#E6EDF5", color: THEME.text }}
              >
                {b.label}
              </a>
            ))}
          </div>
        </div>

        {/* Right bubbles graphic */}
        <BubblesGraphic />
      </div>

      <OfficeGallery />

      {/* Timeline */}
      <div className="mt-10 rounded-2xl border p-8 bg-white" style={{ borderColor: "#E6EDF5" }} data-test="about-timeline">
        <h3 className="text-xl font-bold" style={{ color: THEME.text }}>연혁</h3>
        <ul className="mt-4 space-y-2 text-sm" style={{ color: THEME.textSub }}>
          {history.map((h, i) => (
            <li key={i}><span className="font-semibold" style={{ color: THEME.text }}>{h.year}</span> — {h.text}</li>
          ))}
        </ul>
      </div>
    </section>
  );
}

function BubblesGraphic() {
  return (
    <div className="relative h-72" aria-hidden>
      {/* dotted background */}
      {Array.from({ length: 40 }).map((_, i) => (
        <div
          key={i}
          className="absolute h-2 w-2 rounded-full"
          style={{
            left: `${12 + (i % 8) * 10}%`,
            top: `${10 + Math.floor(i / 8) * 14}%`,
            background: "#E9F0FA",
          }}
        />
      ))}
      {/* main circles */}
      <div
        className="absolute left-[38%] top-[24%] h-24 w-24 rounded-full grid place-items-center text-white"
        style={{ background: THEME.deep, boxShadow: "0 14px 30px rgba(0,26,51,.25)" }}
      >
        UP
      </div>
      <div className="absolute left-[62%] top-[12%] h-12 w-12 rounded-full grid place-items-center text-white" style={{ background: "#6B7A90" }}>
        🛡️
      </div>
      <div className="absolute left-[18%] top-[36%] h-10 w-10 rounded-full grid place-items-center text-white" style={{ background: "#9BB9F3", color: "#0A2540" }}>
        •
      </div>
      <div className="absolute left-[50%] top-[58%] h-12 w-12 rounded-full grid place-items-center text-white" style={{ background: "#FDBA74" }}>
        ↔
      </div>
      <div className="absolute left-[70%] top-[50%] h-12 w-12 rounded-full grid place-items-center text-white" style={{ background: "#CBD5E1", color: "#1f2937" }}>
        ●
      </div>
    </div>
  );
}

function OfficeGallery() {
  // 사용자가 제공한 Google Drive 이미지 링크들 (공유: 링크가 있는 모든 사용자 보기)
  const officeImgs = [
    "https://drive.google.com/file/d/1Y08QnyDvsqXOUMaZn7J8NqFcHEZLO4AG/view?usp=sharing",
    "https://drive.google.com/file/d/1WwuGn2jMOpQ2KoSj-mylsYf5_IAEpuJz/view?usp=sharing",
    "https://drive.google.com/file/d/1ZdbS20hQ7uUXC5vOlPvBX85BdJoamNeo/view?usp=drive_link",
    "https://drive.google.com/file/d/10rdg_TG-zcu7JphTAyUwOO7ab6YE7O4n/view?usp=sharing",
  ];

  const FALLBACK_URL =
    "https://images.unsplash.com/photo-1520607162513-77705c0f0d4a?q=80&w=1200&auto=format&fit=crop";

  const [errorCount, setErrorCount] = React.useState(0);
  const mountedRef = React.useRef(true);
  React.useEffect(() => () => {
    mountedRef.current = false;
  }, []);

  // Drive 링크 → 직접 이미지 URL 후보들 생성 (정규식 리터럴 사용 안 함)
  const buildDriveCandidates = (raw, width = 1600) => {
    try {
      const url = new URL(raw);
      let id = null;
      // 1) /file/d/<id>/view 경로에서 id 추출
      const marker = "/file/d/";
      const idx = url.pathname.indexOf(marker);
      if (idx !== -1) {
        const rest = url.pathname.slice(idx + marker.length);
        id = rest.split("/")[0];
      }
      // 2) 쿼리파라미터 ?id= 보조 추출
      if (!id) id = url.searchParams.get("id");
      const resourcekey = url.searchParams.get("resourcekey");
      if (!id) return [raw];
      const rk = resourcekey ? `&resourcekey=${resourcekey}` : "";

      const baseView = `https://drive.google.com/uc?export=view&id=${id}${rk}`;
      const baseDownload = `https://drive.google.com/uc?export=download&id=${id}${rk}`;
      const basePlain = `https://drive.google.com/uc?id=${id}${rk}`;
      const thumb = `https://drive.google.com/thumbnail?id=${id}&sz=w${width}`;
      // 최후수단: 이미지 프록시를 통한 우회(일부 브라우저/쿠키 정책 환경에서 유용)
      const proxy = `https://images.weserv.nl/?url=${encodeURIComponent(baseView)}`;

      return [baseDownload, baseView, basePlain, thumb, proxy];
    } catch {
      return [raw];
    }
  };

  // tests (간단 유효성)
  React.useEffect(() => {
    console.assert(Array.isArray(officeImgs) && officeImgs.length >= 3, "[TEST] office imgs present");
    try {
      officeImgs.forEach((raw) => {
        const c = buildDriveCandidates(raw);
        console.assert(Array.isArray(c) && c.length >= 3, "[TEST] drive candidates built");
      });
    } catch {}
  }, []);

  return (
    <div className="mt-10" data-test="office-gallery">
      <h3 className="text-xl font-bold" style={{ color: THEME.text }}>전경 사진</h3>
      {errorCount >= officeImgs.length && (
        <div
          className="mt-3 rounded-md border p-3 text-xs"
          style={{ borderColor: "#FECACA", background: "#FFF1F2", color: "#991B1B" }}
        >
          일부 이미지가 로드되지 않았습니다. Google Drive 공유를 <b>링크가 있는 모든 사용자(보기)</b>로 설정했는지 확인해 주세요.
        </div>
      )}
      <div className="mt-4 grid sm:grid-cols-2 gap-4">
        {officeImgs.map((raw, i) => {
          const candidates = buildDriveCandidates(raw);
          const first = candidates[0] || raw;
          return (
            <img
              key={i}
              src={first}
              alt={`히포케이메논 전경 ${i + 1}`}
              className="w-full h-56 object-cover rounded-xl border"
              style={{ borderColor: "#E6EDF5" }}
              referrerPolicy="no-referrer"
              crossOrigin="anonymous"
              data-attempt="0"
              data-candidates={JSON.stringify(candidates)}
              onError={(e) => {
                const img = e?.currentTarget ?? null;
                if (!img) return; // 안전 가드
                // 후보 URL을 순차 시도 → 모두 실패 시 1회만 대체 이미지 적용
                try {
                  const cand = JSON.parse(img.dataset.candidates || "[]");
                  let attempt = parseInt(img.dataset.attempt || "0", 10);
                  if (attempt < cand.length - 1 && !img.dataset.fallbackApplied) {
                    attempt += 1;
                    img.dataset.attempt = String(attempt);
                    img.src = cand[attempt];
                  } else if (!img.dataset.fallbackApplied) {
                    img.dataset.fallbackApplied = "1";
                    img.src = FALLBACK_URL;
                    if (mountedRef.current) setErrorCount((c) => c + 1);
                  }
                } catch {
                  if (!img.dataset.fallbackApplied) {
                    img.dataset.fallbackApplied = "1";
                    img.src = FALLBACK_URL;
                    if (mountedRef.current) setErrorCount((c) => c + 1);
                  }
                }
              }}
            />
          );
        })}
      </div>
    </div>
  );
}

// ===== SERVICES =====
function ServicesPage() {
  const services = [
    {
      id: "wealthy",
      title: "웰시투자그룹",
      subtitle: "차트 분석 거래 기반 투자 교육",
      summary: "비트코인 및 알트코인 차트 분석을 바탕으로 실전 트레이딩 능력을 키우는 교육 프로그램.",
      url: "http://wealthyinvestment.co.kr/",
    },
    {
      id: "tetherback",
      title: "테더백",
      subtitle: "DEX/CEX 매매 수수료 자동 환급 플랫폼",
      summary: "거래소 연동만으로 발생 수수료를 자동 계산·환급. 지갑 연결, 다계정 관리, 정산 리포트 제공.",
      url: "https://tetherback.io/",
    },
    {
      id: "gatu",
      title: "가투법",
      subtitle: "브랜드 본질을 살리는 콘셉트/크리에이티브",
      summary: "콘셉트 도출 → 콘텐츠 실험 → 전환 퍼널 최적화로 성장 설계.",
      url: "https://cafe.naver.com/1djr58",
    },
  ];

  useEffect(() => {
    console.assert(
      document.querySelectorAll('[data-test="service-card"]').length === services.length,
      "[TEST] services cards rendered"
    );
    // Header sanity (추가 테스트)
    const brand = document.getElementById('brand');
    console.assert(!!brand, '[TEST] header brand exists');
  }, []);

  return (
    <section className="mx-auto max-w-7xl px-4 py-20" aria-label="사업분야">
      <h2 className="text-3xl font-extrabold" style={{ color: THEME.text }}>
        사업분야
      </h2>
      <div className="mt-8 grid md:grid-cols-3 gap-6">
        {services.map((s) => (
          <a
            key={s.id}
            data-test="service-card"
            href={s.url}
            target="_blank"
            rel="noreferrer noopener"
            className="text-left rounded-2xl border p-6 bg-white hover:shadow"
            style={{ borderColor: "#E6EDF5" }}
          >
            <div className="text-sm text-slate-500">{s.subtitle}</div>
            <div className="mt-1 text-xl font-bold" style={{ color: THEME.text }}>
              {s.title}
            </div>
            <p className="mt-2 text-sm" style={{ color: THEME.textSub }}>
              {s.summary}
            </p>
            <span className="mt-4 inline-block text-xs font-bold text-slate-600 underline">
              이동하기 ↗
            </span>
          </a>
        ))}
      </div>
    </section>
  );
}

function ServiceDetailPage({ service, onBack }) {
  useEffect(() => {
    console.assert(service && service.title, "[TEST] service detail has data");
  }, [service]);
  if (!service) return null;
  return (
    <section className="mx-auto max-w-7xl px-4 py-20" aria-label="사업 상세">
      <button onClick={onBack} className="mb-6 text-sm underline text-slate-500">
        ← 목록으로
      </button>
      <h2 className="text-3xl font-extrabold" style={{ color: THEME.text }}>
        {service.title}
      </h2>
      <p className="mt-2 text-slate-600">{service.subtitle}</p>
      <div className="mt-6 grid lg:grid-cols-2 gap-8">
        <div className="rounded-2xl border p-6 bg-white" style={{ borderColor: "#E6EDF5" }}>
          <h3 className="text-xl font-bold" style={{ color: THEME.text }}>
            개요
          </h3>
          <p className="mt-2 text-sm" style={{ color: THEME.textSub }}>
            {service.summary}
          </p>
        </div>
        <div className="rounded-2xl border p-6 bg-white" style={{ borderColor: "#E6EDF5" }}>
          <h3 className="text-xl font-bold" style={{ color: THEME.text }}>
            주요 기능
          </h3>
          <ul className="mt-2 list-disc list-inside text-sm" style={{ color: THEME.textSub }}>
            <li>체계적 커리큘럼 / 실시간 피드백</li>
            <li>리스크·손익 관리 모델</li>
            <li>정산 리포트/자동 환급(해당 서비스에 한함)</li>
          </ul>
        </div>
      </div>
    </section>
  );
}

// ===== NEWS =====
function NewsPage({ onOpen }) {
  const posts = [
    {
      id: 1,
      title: "테더백, 2025 한국소비자만족지수 1위",
      date: "2025-02-10",
      url: "https://magazine.hankyung.com/business/article/202501067705b",
      content: "테더백이 2025 한국소비자만족지수 1위를 수상했습니다.",
      thumb: "https://images.unsplash.com/photo-1515378791036-0648a3ef77b2?q=80&w=1200&auto=format&fit=crop",
    },
    {
      id: 2,
      title: "웰시투자그룹, 2025 한국소비자만족지수 1위",
      date: "2025-02-20",
      url: "https://magazine.hankyung.com/business/article/202502104543b",
      content: "웰시투자그룹이 2025 한국소비자만족지수 1위를 받았습니다.",
      thumb: "https://images.unsplash.com/photo-1498050108023-c5249f4df085?q=80&w=1200&auto=format&fit=crop",
    },
    {
      id: 3,
      title: "IBK 기업은행 동반자상 수상",
      date: "2024-09-27",
      url: "https://www.etnews.com/20240927000122",
      content: "IBK 기업은행 동반자상을 수상했습니다.",
      thumb: "https://images.unsplash.com/photo-1522071901873-411886a10004?q=80&w=1200&auto=format&fit=crop",
    },
  ];

  useEffect(() => {
    const anchors = document.querySelectorAll('[data-test="news-link"]');
    console.assert(anchors.length >= posts.length, "[TEST] news links rendered");
    anchors.forEach((a) =>
      console.assert(/^https?:\/\//.test(a.getAttribute("href") || ""), "[TEST] news href valid")
    );
  }, []);

  return (
    <section className="mx-auto max-w-7xl px-4 py-20" aria-label="소식">
      <h2 className="text-3xl font-extrabold" style={{ color: THEME.text }}>
        소식
      </h2>
      <div className="mt-8 grid md:grid-cols-3 gap-6">
        {posts.map((p) => (
          <a
            key={p.id}
            href={p.url}
            target="_blank"
            rel="noreferrer noopener"
            data-test="news-link"
            className="group rounded-2xl border overflow-hidden bg-white"
            style={{ borderColor: "#E6EDF5" }}
          >
            <div
              className="h-40 w-full"
              style={{
                backgroundImage: `url(${p.thumb})`,
                backgroundSize: "cover",
                backgroundPosition: "center",
              }}
            />
            <div className="p-4">
              <div className="text-sm text-slate-500">{p.date}</div>
              <div className="mt-1 font-bold group-hover:underline" style={{ color: THEME.text }}>
                {p.title}
              </div>
            </div>
          </a>
        ))}
      </div>
    </section>
  );
}

function NewsDetailPage({ post, onBack }) {
  if (!post) return null;
  return (
    <section className="mx-auto max-w-7xl px-4 py-20" aria-label="소식 상세">
      <button onClick={onBack} className="mb-6 text-sm underline text-slate-500">
        ← 목록으로
      </button>
      <div className="max-w-3xl">
        <h2 className="text-3xl font-extrabold" style={{ color: THEME.text }}>
          {post.title}
        </h2>
        <div className="mt-2 text-sm text-slate-500">{post.date}</div>
        <p className="mt-6 text-base leading-7" style={{ color: THEME.textSub }}>
          {post.content}
        </p>
        <a href={post.url} target="_blank" rel="noreferrer noopener" className="mt-6 inline-block text-sm font-bold underline text-slate-600">
          원문 보기 ↗
        </a>
      </div>
    </section>
  );
}

// ===== CONTACT (only "오시는 길") =====
function ContactPage() {
  const NAVER_MAP_URL =
    "https://map.naver.com/p/search/%ED%9E%88%ED%8F%AC%EC%BC%80%EC%9D%B4%EB%A9%94%EB%85%BC/place/1908105800";

  useEffect(() => {
    const link = document.querySelector('[data-test="naver-map"]');
    console.assert(!!link, "[TEST] naver map link present");
  }, []);

  return (
    <section className="mx-auto max-w-7xl px-4 py-20" aria-label="안내">
      <div className="rounded-2xl border p-6 bg-white" style={{ borderColor: "#E6EDF5" }}>
        <div className="flex items-center justify-between">
          <h2 className="text-2xl font-extrabold" style={{ color: THEME.text }}>
            오시는 길
          </h2>
          <a
            data-test="naver-map"
            href={NAVER_MAP_URL}
            target="_blank"
            rel="noreferrer noopener"
            className="rounded-full px-4 py-2 text-xs font-bold border"
            style={{ borderColor: THEME.light, color: THEME.light }}
          >
            네이버 지도 ↗
          </a>
        </div>
        <div className="mt-4 text-sm" style={{ color: THEME.textSub }}>
          서울특별시 마포구 월드컵북로2길 57, 4·5층 (동교동, 금화빌딩)
          <br />지하철: 2호선 홍대입구역 2번 출구 1분
        </div>
      </div>
    </section>
  );
}

// ===== FOOTER =====
function Footer({ setRoute }) {
  return (
    <footer className="mt-20 border-t" style={{ borderColor: "#E6EDF5", background: THEME.bg }}>
      <div className="mx-auto max-w-7xl px-4 py-12 grid sm:grid-cols-2 gap-6 items-center">
        <div>
          <div className="flex items-center gap-2">
            <div className="h-8 w-8 rounded grid place-items-center text-white" style={{ background: THEME.deep2 }}>
              <Globe2 className="h-4 w-4" />
            </div>
            <div className="font-bold" style={{ color: THEME.text }}>
              HYPOKEIMENON
            </div>
          </div>
          <p className="mt-3 text-xs text-slate-500">
            © {new Date().getFullYear()} 히포케이메논. All rights reserved.
          </p>
        </div>
        <div className="justify-self-end flex items-center gap-4 text-sm">
          {NAV.map((n) => (
            <button
              key={n.id}
              onClick={() => setRoute(n.id)}
              className="hover:underline"
              style={{ color: THEME.textSub }}
            >
              {n.label}
            </button>
          ))}
        </div>
      </div>
    </footer>
  );
}
