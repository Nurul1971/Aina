/* =====================================================
   Blogger Page Navigation (Modern Version)
   No document.write | No inline onclick | Fast & Clean
===================================================== */

(() => {
  'use strict';

  const state = {
    page: 1,
    total: 1,
    label: null,
    type: 'page'
  };

  /* ===============================
     Utility
  =============================== */

  const qs = name => new URLSearchParams(location.search).get(name);

  const getPageFromHash = () => {
    const m = location.hash.match(/PageNo=(\d+)/);
    return m ? parseInt(m[1], 10) : 1;
  };

  const fetchJSONP = (url, callback) => {
    const s = document.createElement('script');
    const cb = `jsonp_${Date.now()}`;
    window[cb] = data => {
      callback(data);
      delete window[cb];
      s.remove();
    };
    s.src = `${url}&callback=${cb}`;
    document.head.appendChild(s);
  };

  /* ===============================
     Init
  =============================== */

  const init = () => {
    state.page = getPageFromHash();
    detectType();
    loadTotalPosts();
  };

  const detectType = () => {
    const path = location.pathname;
    if (path.includes('/search/label/')) {
      state.type = 'label';
      state.label = decodeURIComponent(
        path.split('/search/label/')[1].split('?')[0]
      );
    }
  };

  /* ===============================
     Load total posts
  =============================== */

  const loadTotalPosts = () => {
    const url = state.type === 'label'
      ? `${home_page}feeds/posts/summary/-/${state.label}?alt=json-in-script&max-results=1`
      : `${home_page}feeds/posts/summary?alt=json-in-script&max-results=1`;

    fetchJSONP(url, data => {
      state.total = Math.ceil(
        parseInt(data.feed.openSearch$totalResults.$t, 10) / postperpage
      );
      renderPager();
    });
  };

  /* ===============================
     Render Pager
  =============================== */

  const renderPager = () => {
    const containers = document.querySelectorAll('[name="pageArea"], #blog-pager');
    if (!containers.length) return;

    const nav = document.createElement('nav');
    nav.className = 'pagination';

    const start = Math.max(1, state.page - numshowpage);
    const end = Math.min(state.total, state.page + numshowpage);

    nav.appendChild(pageInfo());

    if (state.page > 1) {
      nav.appendChild(pageLink(state.page - 1, upPageWord));
    }

    for (let i = start; i <= end; i++) {
      nav.appendChild(
        i === state.page
          ? currentPage(i)
          : pageLink(i, i)
      );
    }

    if (state.page < state.total) {
      nav.appendChild(pageLink(state.page + 1, downPageWord));
    }

    containers.forEach(el => {
      el.innerHTML = '';
      el.appendChild(nav.cloneNode(true));
    });
  };

  /* ===============================
     Elements
  =============================== */

  const pageInfo = () => {
    const s = document.createElement('span');
    s.className = 'showpageOf';
    s.textContent = `Page ${state.page} of ${state.total}`;
    return s;
  };

  const currentPage = n => {
    const s = document.createElement('span');
    s.className = 'showpagePoint';
    s.textContent = n;
    return s;
  };

  const pageLink = (page, text) => {
    const a = document.createElement('a');
    a.href = buildUrl(page);
    a.textContent = text;
    a.className = 'showpageNum';
    return a;
  };

  /* ===============================
     URL Builder (SEO Friendly)
  =============================== */

  const buildUrl = page => {
    const base = state.type === 'label'
      ? `/search/label/${state.label}`
      : `/search`;

    return `${base}?max-results=${postperpage}#PageNo=${page}`;
  };

  document.addEventListener('DOMContentLoaded', init);

})();
