---
title: Interacting with map
date: 2025-07-11
category: 'react project'
---

记住状态，

```react
  useEffect(() => {
    if (mapLat && mapLng) setMapPosition([mapLat, mapLng]);
  }, [mapLat, mapLng]);
```

重命名 把isLoading重命名成isLoadingPosition

```react
  const {
    isLoading: isLoadingPosition,
    position: geoLocationPosition,
    getPosition,
  } = useGeolocation();
```
