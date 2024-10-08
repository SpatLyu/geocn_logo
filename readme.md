
``` r
# devtools::install('SpatLyu/geocn')

library(sf)
library(ggplot2)
library(geocn)

province = load_cn_province()
tenline = load_cn_tenline()
cn_border = load_cn_border()
load_cn_landcoast() |> 
  st_centroid() |> 
  st_coordinates() -> center
lon = center[1]
lat = center[2]

ortho= paste0('+proj=ortho +lat_0=', lat, ' +lon_0=', lon,
              ' +x_0=0 +y_0=0 +a=6371000 +b=6371000 +units=m +no_defs')
circle = suppressMessages(
  sf::st_point(x = c(0, 0)) %>%
    sf::st_buffer(dist = 3.8e6) %>%
    sf::st_sfc(crs = ortho)) 
world = load_world_coastline() |> 
  st_transform(st_crs(ortho)) |> 
  st_intersection(circle)

ggplot() +
  geom_sf(data = circle,colour="black",fill="transparent") +
  geom_sf(data = world,fill="grey80",colour="grey80") +
  geom_sf(data = province,fill = "transparent",size=.1,color="grey") + 
  geom_sf(data = cn_border,size = 1.2,color="red") +
  ggfx::with_outer_glow(geom_sf(data = cn_border,
                                lwd = .3,
                                color = 'red'),
                        colour = "#9d98b7",
                        sigma = 5,
                        expand = 3.5) +
  coord_sf(crs = ortho) +
  theme_void() -> cn

ggsave(filename = './cn.png',plot = cn,
       dpi = 120,bg = 'transparent',
       width = 3.2,height = 3.2)

library(showtext)
showtext_auto(enable = TRUE)
font_add("ShineTypewriter", regular = "./ShineTypewriter-lgwzd.ttf")
library(hexSticker)
library(magick)

sticker(
  subplot = "./cn.png",
  s_x = .995,
  s_y = 1.02,
  s_width = .75,
  s_height = .75,
  package = "geocn",
  p_family = "ShineTypewriter",
  p_size = 25,
  p_color = ggplot2::alpha("#3e3221",.9),
  p_x = 1.02,
  p_y = 1.01,
  dpi = 300,
  asp = 1,
  h_size = 1.55,
  h_color = ggplot2::alpha("#3e3221",.75),
  h_fill = '#ffffff',
  white_around_sticker = F,
  url = "https://stscl.github.io/geocn",
  u_color = ggplot2::alpha("#3e3221",1),
  u_size = 5.25,
  filename = "geocn_logo.png"
)

image_read('./geocn_logo.png') |> 
  image_resize("240x278")|> 
  image_write('./geocn_logo.png')
```

![](./geocn_logo.png)
