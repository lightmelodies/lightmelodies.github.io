---
title: "Pixiv Crawler"
---

{% highlight javascript %}
function crawler(page: number) {
  request({ url: encodeURI(pixiv + `?p=${page}`) }, (_e, _r, body) => {
    let json = JSON.parse(body);
    let illusts: Illust[] = json.body.illust.data.filter((x: Illust) => x.url);
    illusts.forEach((illust) => {
      let url = illust.url
        .replace(
          /\/c\/250x250_80_a2\/img-master(.*?)_square1200/,
          "/img-original$1"
        )
        .replace(
          /\/c\/250x250_80_a2\/custom-thumb(.*?)_custom1200/,
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
{% endhighlight %}