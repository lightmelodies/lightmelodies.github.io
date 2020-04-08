---
title: "Pixiv Crawler"
---

{% highlight typescript %}
import request from "request";
import fs from "fs";

const pixiv =
  "https://www.pixiv.net/ajax/search/illustrations/初音ミク 30000users入り";

const cookie = "...";

interface Illust {
  illustId: string;
  illustTitle: string;
  userId: string;
  userName: string;
  url: string;
}

function crawler(page: number) {
  request({ url: encodeURI(pixiv + `?p=${page}`) }, (_e, _r, body) => {
    let json = JSON.parse(body);
    let illusts: Illust[] = json.body.illust.data.filter((x: Illust) => x.url);
    illusts.forEach((illust) => {
      let url = illust.url
        .replace(
          /\/c\/250x250_80_a2\/img-master(.*?)_square1200/,
          "/img-original$1"
        );
      const req = request({
        url: url,
        headers: {
          cookie: cookie,
          referer: "https://www.pixiv.net/artworks/" + illust.illustId,
        },
      }).on("response", function (res) {
        if (res.statusCode === 200) {
          console.log(url);
          req.pipe(fs.createWriteStream("pixiv/" + url.split("/")[11]));
        } else {
          console.log(url, res.statusMessage);
        }
      });
    });
  });
}

request({ url: encodeURI(pixiv) }, (_e, _r, body) => {
  let total = JSON.parse(body).body.illust.total;
  console.log(total);
  for (let i = 1; i < total / 60; i++) {
    crawler(i);
  }
});

{% endhighlight %}