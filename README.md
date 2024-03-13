# 微信历史图片批量转换为jpg，nodejs版本

 
//修改图片//修改图片
let fs = require("fs");
let path = require("path");
let async = require("async"); //值是多少自己算。
let base = 0xff;
let next = 0xd8;
let gifA = 0x47;
let gifB = 0x49;
let pngA = 0x89;
let pngB = 0x50;
let scanDir = "D:/WeChat Files/wxid_xxxxx/FileStorage/Image";
let imgDir = "D:/aa";
let files = fs.readdirSync(scanDir);
var arr = [];

files.forEach(function (ml) {
  if (ml === "Thumb") return;
  let _cfiles = fs.readdirSync(scanDir + "/" + ml);
  _cfiles.forEach((item) => {
    if (path.extname(item) == ".dat") {
      arr.push("/" + ml + "/" + item);
    }
  });
});
async.mapLimit(
  arr,
  50,
  function (item, cb) {
    convert(item, cb);
  },
  function () {
    process.exit(0);
  }
);
//convert
function convert(item, cb) {
  console.log(item);
  const lastLen = item.lastIndexOf("/");
  const ml = item.slice(0, lastLen);
  const file = item.slice(lastLen + 1);
  let absPath = path.join(scanDir + ml, file);
  let imgPath = path.join(imgDir + ml, file + ".jpg");

  const fileSave = function () {
    fs.readFile(absPath, (err, content) => {
      if (err) {
        console.log(err);
        cb(err);
      }
      let firstV = content[0],
        nextV = content[1],
        jT = firstV ^ base,
        jB = nextV ^ next,
        gT = firstV ^ gifA,
        gB = nextV ^ gifB,
        pT = firstV ^ pngA,
        pB = nextV ^ pngB;
      var v = firstV ^ base;
      if (jT == jB) {
        v = jT;
      } else if (gT == gB) {
        v = gT;
      } else if (pT == pB) {
        v = pT;
      }
      let bb = content.map((br) => {
        return br ^ v;
      });
      fs.writeFileSync(imgPath, bb);
      cb(null);
    });
  };
  // 异步检查目录是否存在
  fs.access(imgDir + ml, fs.constants.F_OK, (err) => {
    if (err) {
      // 目录不存在，创建目录
      fs.mkdir(imgDir + ml, { recursive: true }, (mkdirErr) => {
        if (mkdirErr) {
          console.error(mkdirErr);
        } else {
          console.log("Directory created successfully");
          fileSave();
        }
      });
    } else {
      //   console.log("Directory already exists");
      fileSave();
    }
  });
}
