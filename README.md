# Containerlab Topology Visualization with NextUI

![containerlab_topo_with_nextui](https://raw.githubusercontent.com/ahmedalodhari/Containerlab_NextUI_Template/main/containerlab_topo_with_nextui.gif)

A NextUI-based HTML template to visualize network topologies that were built by Containerlab.

## Stack 

- [Containerlab](https://containerlab.srlinux.dev/)
- [NextUI](https://developer.cisco.com/site/neXt/)
- [Tailwind CSS](https://tailwindcss.com/)

## Usage

##### 1. Specify the role of each node using the `group` configuration attribute of Containerlab

The `group` attribute defines the role of each node. This value is used for the on-demand horizontal/vertical layout of the topology.

An example of a Containerlab YAML configuration file that leverages the `group` attribute is shown below.

```yaml
name: frr01

topology:
  nodes:
    spine:
      kind: linux
      image: frrouting/frr
      group: spine
      binds:
        - spine/daemons:/etc/frr/daemons
    leaf1:
      kind: linux
      image: frrouting/frr
      group: leaf
      binds:
        - leaf1/daemons:/etc/frr/daemons
    leaf2:
      kind: linux
      image: frrouting/frr
      group: leaf
      binds:
        - leaf2/daemons:/etc/frr/daemons
    client1:
      kind: linux
      image: ghcr.io/hellt/network-multitool
      group: server
    client2:
      kind: linux
      image: ghcr.io/hellt/network-multitool
      group: server

  links:
    - endpoints: ["spine:eth1", "leaf1:eth1"]
    - endpoints: ["spine:eth2", "leaf2:eth1"]
    - endpoints: ["leaf1:eth2", "client1:eth1"]
    - endpoints: ["leaf2:eth2", "client2:eth1"] 
```

The horizontal/vertical sort order of the nodes can be controlled by the `sortOrder` parameter defined in the `clab_blue_label_template.js` file.

```javascript
layoutConfig: {
  sortOrder: ['superspine', 'spine', 'leaf', 'server'],
},
```

##### 2. Serve the static files linked by the HTML template

The static files required by the HTML template are: 

- NextUI stylesheet `next.css` & fonts
- Template stylesheet `clab_style.css`
- NextUI JS file `next.js` 
- Template topology JS file `clab_blue_label_template.js`

The Containerlab [embedded web server](https://pkg.go.dev/net/http) isn't programmed to serve static files. It only serves a default HTML template or a custom one via the `--template` flag. So, to serve the static files needed, we have to use an external web server. 

Alternatively, we can add this capability to Containerlab by modifying the `graph` code as shown [here](https://github.com/srl-labs/containerlab/compare/main...ahmedalodhari:graph-serve-static-files). 

Using Containerlab to serve the static files is advantages verses using an external web server because we don't have to worry about or waste time configuring/troubleshooting another tool, and instead focus on building and visualizing our networks directly from Containerlab.  

##### 3. Generate the topology

Generate the network topology by indicating the custom HTML file with the `--template` flag

```bash
containerlab graph --topo topo.clab.yml --template clab_blue_label_template.html 
```

If using the modified [Containerlab](https://github.com/srl-labs/containerlab/compare/main...ahmedalodhari:graph-serve-static-files), then we need to also specify the directory where the static files are served from

```bash
containerlab graph --topo topo.clab.yml --template clab_blue_label_template.html --static-dir ./assets
```

##### 4.  Enjoy

The topology can be access by default at `http://<docker-host-ip>:50080`

## Known limitations

- The interface labels will overlap each other when viewing large topology on smaller screens.
- The enter-full-screen mode doesn't work.
