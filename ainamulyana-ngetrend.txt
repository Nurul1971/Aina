(function () {
  "use strict";

  /* ===== Ambil setting dari blog ===== */
  const PER_PAGE = window.postperpage || 6;
  const SHOW_PAGE = window.numshowpage || 2;
  const PREV_TEXT = window.upPageWord || "Prev";
  const NEXT_TEXT = window.downPageWord || "Next";

  const pager = document.getElementById("blog-pager") ||
                document.querySelector("[name='pageArea']") ||
                document.getElementById("pagination");

  if (!pager) return;

  function getPage() {
    const m = location.hash.match(/PageNo=(\d+)/);
    return m ? parseInt(m[1], 10) : 1;
  }

  function getLabel() {
    const m = location.pathname.match(/\/search\/label\/([^/?]+)/);
    return m ? decodeURIComponent(m[1]) : null;
  }

  async function getTotal(label) {
    const url = label
      ? `/feeds/posts/summary/-/${label}?alt=json&max-results=0`
      : `/feeds/posts/summary?alt=json&max-results=0`;

    const r = await fetch(url);
    const j = await r.json();
    return parseInt(j.feed.openSearch$totalResults.$t, 10);
  }

  function pageLink(page, label) {
    if (page === 1) {
      return label
        ? `/search/label/${label}?max-results=${PER_PAGE}`
        : `/?max-results=${PER_PAGE}`;
    }
    return `#PageNo=${page}`;
  }

  function render(current, total, label) {
    let html = `<span class="page-info">Page ${current} of ${total}</span>`;

    if (current > 1) {
      html += `<a href="${pageLink(current - 1, label)}" rel="prev">${PREV_TEXT}</a>`;
    }

    const start = Math.max(1, current - SHOW_PAGE);
    const end = Math.min(total, current + SHOW_PAGE);

    for (let i = start; i <= end; i++) {
      html += i === current
        ? `<span class="current">${i}</span>`
        : `<a href="${pageLink(i, label)}">${i}</a>`;
    }

    if (current < total) {
      html += `<a href="${pageLink(current + 1, label)}" rel="next">${NEXT_TEXT}</a>`;
    }

    pager.innerHTML = html;
  }

  (async function init() {
    try {
      const label = getLabel();
      const current = getPage();
      const totalPost = await getTotal(label);
      const totalPage = Math.ceil(totalPost / PER_PAGE);

      if (totalPage > 1) {
        render(current, totalPage, label);
      }
    } catch (e) {
      console.warn("Pagination disabled:", e);
    }
  })();

})();
